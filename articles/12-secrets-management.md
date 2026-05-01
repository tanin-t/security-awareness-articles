# บทความที่ 12: Secrets Management — อย่า Hardcode ความลับลงใน Code

**CyberSecurity Awareness Series — Part 5: Developer-Specific**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของ AWS Bill ที่กลายเป็น $50,000 ในคืนเดียว

เดือนมีนาคม ปี 2014 dev คนหนึ่งชื่อ **Joe Moreno** post code ของตัวเองบน GitHub เป็น public repo — ในนั้นมีไฟล์ `config.py` ที่มี AWS credentials ของเขาอยู่:

```python
AWS_ACCESS_KEY_ID = "<your-access-key>"     # จริง อยู่ใน code
AWS_SECRET_ACCESS_KEY = "<your-secret>"      # จริง อยู่ใน code
```

ภายใน **5 นาที** หลัง push — AWS account ของเขาถูกเข้าถึง  
ภายใน **30 นาที** — มี EC2 instance type **c4.8xlarge** (เครื่องราคาแพง) **30 ตัว** ถูก launch ใน region ที่เขาไม่ได้ใช้  
ภายใน **2 ชั่วโมง** — อีก 100+ instances ใน region อื่นๆ  
ตอนเช้าเขาตื่นมา — **bill $50,000** ที่ไม่เคยตั้งใจ

ผู้โจมตีใช้ instances เหล่านี้ขุด cryptocurrency — ทำเงินให้เขาแค่ไม่กี่พันดอลลาร์ แต่ Joe ต้องจ่าย $50,000

[อ่านเรื่องนี้ที่ blog ของ Joe Moreno](https://www.applausible.com/blog/2014/05/04/i-pushed-credentials-to-github)

ที่น่าตกใจที่สุด — **มีเครื่องมืออัตโนมัติ** ที่ scan GitHub ตลอดเวลา หา API keys ที่หลุด ใช้เวลาน้อยกว่า 5 นาทีนับจากที่ commit เข้า public repo

ในปี 2026 — ปัญหานี้ใหญ่ขึ้นมาก:

- **GitGuardian** รายงานในปี 2024 ว่า **23 ล้าน secrets ถูก leak ใน GitHub public repos** ในปีนั้นเดียว ([รายงาน State of Secrets Sprawl](https://www.gitguardian.com/state-of-secrets-sprawl-report-2024))
- **AWS, GCP, Azure** ทั้งหมดมี **automatic key revocation** เมื่อ detect ว่า key อยู่ใน public repo (ดี — แต่ก็ไม่ได้ revoke ก่อนที่ผู้โจมตีจะใช้)
- ผู้โจมตีมี **botnet ที่ scan GitHub** — keys ที่ commit ไป มักโดน abuse ใน **38 วินาที** average ([รายงานของ Truffle Security](https://trufflesecurity.com))

นี่คือเรื่องที่เราจะแก้กันในบทความนี้ — **secrets management** สำหรับ developer

---

## ทำไมการ Hardcode Secrets ถึงเป็นปัญหาใหญ่

### ปัญหาที่หลายคนไม่เห็น

หลายคนคิดว่า:

> "ฉันเก็บ secret ใน private repo — ปลอดภัยแน่นอน"

**ผิด** ครับ เพราะ:

#### 1. Repo อาจเปลี่ยนเป็น public โดยไม่ตั้งใจ

- ผู้พัฒนาเปลี่ยน visibility ผิด
- Forking → ใครก็เห็นได้
- Open-sourcing — ลืม clean history

#### 2. Pull จาก private → push public

```bash
git clone git@github.com:company/private-repo.git
# ทำงานเสร็จ → push ไป personal public fork
git remote set-url origin git@github.com:user/personal-repo.git
git push -u origin main
# 💥 secret หลุด
```

#### 3. Account ของ collaborator โดนแฮก

ใครก็ตามที่มีสิทธิ์เห็น repo (collaborators, organization members, contractors) → account เขาถูกแฮก = secrets ของคุณรั่ว

#### 4. Repo Mirror / Backup รั่ว

หลายบริษัท mirror code ไปที่อื่นโดยไม่ track — secret ก็ตามไปด้วย

#### 5. Build Artifacts รั่ว

```dockerfile
FROM python:3.11
COPY . /app
ENV API_KEY="<your-api-key>"  # ❌ อยู่ใน image
```

Docker image ที่ push ไป registry → คนเปิด layer ดูได้

#### 6. Logs

ถ้า secret อยู่ในตัวแปรที่ถูก log:

```python
print(f"Connecting with config: {config}")
# config มี API_KEY
```

→ logs ที่ส่งไป CloudWatch / Datadog / ELK = secret รั่ว

#### 7. Error Messages

```python
try:
    api_call(API_KEY)
except Exception as e:
    raise Exception(f"Failed: {e}")  # อาจ include secret ใน traceback
```

#### 8. Source Maps / Bundles

**Frontend:**
```javascript
// React app ที่ build แล้ว — secret อยู่ใน bundle.js
const API_KEY = "<your-api-key>";  // ❌ ทุก user เห็นใน DevTools
```

> **กฎเหล็ก: ห้ามมี secret อยู่ใน frontend code เลย** — ทุกอย่างใน frontend คน user เห็นได้

### History ของ Git ก็ยัง Track อยู่

แม้คุณ remove secret ใน commit ใหม่ — **history ยังมี secret อยู่**:

```bash
# เพิ่ม secret โดยไม่ตั้งใจ
git add config.py
git commit -m "add config"
git push

# ตื่นได้คิด ลบออก
git rm config.py  
git commit -m "remove config"
git push

# ❗ ใน git history ยังมี secret อยู่ — ใครก็ git checkout มาได้
```

ต้อง rewrite history (rebase / filter-branch) — ซึ่งมีความเสี่ยงเอง + ต้อง force push (หลายคนทำพลาด)

> **เคล็ดลับ: ถ้า secret หลุด — `revoke` ทันที สำคัญกว่า rewrite history**

ไปที่ provider → revoke key เก่า → generate ใหม่ → ใช้ใหม่  
secret ใน git history ก็ไม่มีค่าอีกแล้ว

---

## พื้นฐานที่ต้องทำ — `.env` + `.gitignore`

### Step 1: ใช้ Environment Variables

แทนที่จะ hardcode → load จาก env:

```python
# ❌ WRONG
API_KEY = "<your-stripe-secret-key>"

# ✅ CORRECT
import os
API_KEY = os.environ['STRIPE_SECRET_KEY']
```

```javascript
// ❌ WRONG
const apiKey = "<your-api-key>";

// ✅ CORRECT
const apiKey = process.env.API_KEY;
```

### Step 2: ใช้ `.env` File ใน Local Dev

```bash
# .env (ใน root ของ project)
STRIPE_SECRET_KEY=<your-stripe-test-key>
DATABASE_URL=postgresql://localhost:5432/mydb
JWT_SECRET=<random-secret>
```

```python
# Python — ใช้ python-dotenv
from dotenv import load_dotenv
load_dotenv()

API_KEY = os.environ['STRIPE_SECRET_KEY']
```

```javascript
// Node — ใช้ dotenv
require('dotenv').config();

const apiKey = process.env.STRIPE_SECRET_KEY;
```

### Step 3: เพิ่ม `.env` เข้า `.gitignore`

```
# .gitignore
.env
.env.local
.env.*.local
.env.production
.env.development

# ห้าม commit secret files
secrets.yml
credentials.json
config.private.*
*.pem
*.key
```

### Step 4: ใช้ `.env.example` หรือ `.env.sample`

ไฟล์ที่บอก dev ใหม่ว่าต้องตั้ง env อะไรบ้าง — **ไม่มี value จริง**:

```bash
# .env.example (commit ได้)
STRIPE_SECRET_KEY=
DATABASE_URL=
JWT_SECRET=
SENDGRID_API_KEY=
```

Dev ใหม่ทำ `cp .env.example .env` แล้วใส่ value เอง

### Step 5: Document ใน README

```markdown
## Setup

1. Copy `.env.example` to `.env`
2. Get Stripe API key from team lead (1Password vault)
3. Generate JWT_SECRET: `openssl rand -hex 32`
```

---

## แต่... `.env` File ไม่พอ

`.env` ดีสำหรับ local dev แต่ **production** มีปัญหาอื่น:

### ปัญหาของ `.env` ใน Production

#### 1. แชร์ระหว่างทีม

- Email .env file → email server เก็บ
- Slack DM → log ใน workspace
- Drop ใน shared drive → ใครก็เห็น

→ ต้องการ **centralized place** ที่ทุกคนเข้าถึงตามสิทธิ์

#### 2. Version Control ของ Secrets

- มี secret v1 เก่า ที่ revoke แล้ว
- มี secret v2 ใหม่ ที่ใช้อยู่
- ใครเปลี่ยนเมื่อไหร่?

→ ต้องการ **audit log** + **versioning**

#### 3. Rotation

- ทุก 90 วันต้องเปลี่ยน — ใครจะแก้ทุกเครื่องที่ deploy?

→ ต้องการ **dynamic secret retrieval** ที่ apps ดึงล่าสุดเสมอ

#### 4. Granular Access Control

- Service A ต้องใช้ API key X
- Service B ต้องใช้ API key Y
- แต่ละ service เห็นเฉพาะ secret ที่ตัวเองต้องการ

→ ต้องการ **per-secret access policy**

#### 5. Encryption at Rest

- `.env` file ใน disk ของ server = plaintext

→ ต้องการ **encryption** ใน vault

### Solution: Secret Management Tools

---

## Secret Management Tools

### ระดับ 1: Cloud-native Secret Manager

ถ้าคุณใช้ cloud provider — เริ่มที่นี่ก่อน:

#### AWS Secrets Manager

```python
import boto3
import json

client = boto3.client('secretsmanager', region_name='us-east-1')
response = client.get_secret_value(SecretId='prod/myapp/db-credentials')
secret = json.loads(response['SecretString'])

db_password = secret['password']
```

**Pros:**
- Built-in กับ AWS
- IAM-based access control
- Automatic rotation (built-in สำหรับ RDS)
- Audit log via CloudTrail
- Versioning

**Pricing:** $0.40/secret/month + $0.05 per 10K API calls

#### GCP Secret Manager

```python
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()
name = f"projects/my-project/secrets/api-key/versions/latest"
response = client.access_secret_version(request={"name": name})
secret = response.payload.data.decode("UTF-8")
```

**Pros:**
- IAM-based
- Audit logs
- Versioning
- Integrated กับ GCP services

**Pricing:** $0.06/secret/month + $0.03 per 10K accesses

#### Azure Key Vault

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://myvault.vault.azure.net/", credential=credential)
secret = client.get_secret("api-key")
api_key = secret.value
```

**Pros:**
- Azure AD integration
- Hardware-backed (HSM option)
- Auto-rotation

#### Doppler

Multi-cloud, dev-friendly secret manager:

```bash
# ติดตั้ง
brew install dopplerhq/cli/doppler

# Login
doppler login

# Setup project
doppler setup

# Run app กับ secrets
doppler run -- npm start
```

**Pros:**
- Best DX (developer experience)
- Multi-environment (dev/staging/prod)
- Sync ไปยัง multiple services
- Web UI ดี

### ระดับ 2: Self-hosted Vault

#### HashiCorp Vault

ตัวที่ enterprise ใช้กันมากที่สุด:

```bash
# Login
vault login

# เก็บ secret
vault kv put secret/myapp/db password=<your-password> user=admin

# อ่าน secret
vault kv get secret/myapp/db
```

```python
# Python client
import hvac

client = hvac.Client(url='https://vault.company.com', token='<your-vault-token>')
secret = client.secrets.kv.read_secret_version(path='myapp/db')
db_password = secret['data']['data']['password']
```

**Pros:**
- ความสามารถมหาศาล:
  - Static secrets
  - Dynamic secrets (database, AWS, etc. — generated on-demand, expire automatically)
  - PKI (issue certificates)
  - Encryption-as-a-Service
  - SSH signing
- Self-hosted — full control
- Audit log ละเอียด
- HA, DR support

**Cons:**
- Setup ซับซ้อน
- ต้องการ ops team
- เหมาะกับ enterprise มากกว่า startup

#### OpenBao

Fork ของ HashiCorp Vault หลังจาก Vault เปลี่ยน license ในปี 2023 — open source 100%:

[openbao.org](https://openbao.org)

ถ้าต้องการ self-hosted ที่ open source — OpenBao เป็นตัวเลือก

### ระดับ 3: Kubernetes-native

#### Sealed Secrets (Bitnami)

Encrypt Kubernetes secrets แล้ว commit ลง Git ได้:

```bash
# Encrypt secret
echo -n "<your-password>" | kubectl create secret generic db --dry-run=client \
  --from-file=password=/dev/stdin -o yaml | \
  kubeseal --format=yaml > sealed-secret.yaml

# commit sealed-secret.yaml ลง Git ได้! 
# (เฉพาะ controller ที่มี private key ถึง decrypt ได้)
```

#### External Secrets Operator

Sync จาก external secret managers (AWS, GCP, Vault) → Kubernetes secrets:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: db-credentials
spec:
  secretStoreRef:
    name: aws-secrets-manager
  target:
    name: db-secret
  data:
    - secretKey: password
      remoteRef:
        key: prod/myapp/db-credentials
        property: password
```

#### SOPS (Mozilla)

เข้ารหัสไฟล์ YAML/JSON ด้วย AWS KMS / GCP KMS / age:

```bash
# Encrypt
sops --encrypt --aws-kms <kms-key-arn> secrets.yaml > secrets.enc.yaml

# commit secrets.enc.yaml → ใครก็อ่านไม่ได้ถ้าไม่มี KMS key

# Decrypt ตอนใช้
sops --decrypt secrets.enc.yaml
```

### Comparison Matrix

| Tool | Best For | Self-hosted | Cost |
|---|---|---|---|
| AWS Secrets Manager | AWS shops | No | $$ |
| GCP Secret Manager | GCP shops | No | $ |
| Azure Key Vault | Azure shops | No | $ |
| Doppler | Multi-cloud, startup | No | $ (free tier) |
| HashiCorp Vault | Enterprise, complex needs | Yes | $$$ (Enterprise) / Free (OSS) |
| OpenBao | Vault but open source | Yes | Free |
| Sealed Secrets | k8s-only | Yes | Free |
| SOPS | GitOps-friendly | Yes | Free |

### คำแนะนำของผม

- **Solo dev / startup early-stage:** ใช้ cloud-native (AWS Secrets Manager / GCP Secret Manager) หรือ Doppler
- **Mid-size company:** Doppler หรือ cloud-native
- **Enterprise:** HashiCorp Vault หรือ cloud-native ผสมกัน
- **GitOps shop:** SOPS + cloud KMS
- **k8s-heavy:** External Secrets Operator + cloud-native backend

---

## Pre-commit Hooks — กันก่อนที่ Secret จะหลุด

ป้องกันที่ดีที่สุดคือ **อย่าให้ secret push ขึ้น repo เลย**

### Tools ที่ใช้กัน

#### 1. Gitleaks

[gitleaks.io](https://gitleaks.io)

```bash
# ติดตั้ง
brew install gitleaks

# Scan repo ปัจจุบัน
gitleaks detect --source . --verbose

# Scan history ทั้งหมด
gitleaks detect --source . --log-opts="--all"

# Scan single commit
gitleaks detect --source . --log-opts="abc123"
```

#### 2. TruffleHog

[trufflesecurity.com](https://trufflesecurity.com)

```bash
# Scan local repo
trufflehog filesystem .

# Scan GitHub repo
trufflehog github --repo=https://github.com/owner/repo

# Verify secrets (ทดสอบว่า key ใช้ได้จริงไหม)
trufflehog filesystem . --only-verified
```

TruffleHog มีจุดเด่นที่ **verify** — ลอง call API ด้วย key ที่เจอ → ถ้า work = key ของจริง (severity สูง)

#### 3. detect-secrets (Yelp)

```bash
# ติดตั้ง
pip install detect-secrets

# Generate baseline
detect-secrets scan > .secrets.baseline

# Commit baseline ลง repo
git add .secrets.baseline

# ใน CI — เช็คว่ามี secret ใหม่ไหม
detect-secrets scan --baseline .secrets.baseline
```

### Pre-commit Hook Setup

ติดตั้ง [pre-commit](https://pre-commit.com) framework:

```bash
pip install pre-commit
```

สร้าง `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
  
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
        args: ['--baseline', '.secrets.baseline']
  
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: detect-private-key
      - id: detect-aws-credentials
```

ติดตั้ง hook:

```bash
pre-commit install
```

ตั้งแต่นี้ — ทุก `git commit` จะ scan ก่อน push อัตโนมัติ

### CI/CD Integration

ใน GitHub Actions:

```yaml
name: Secret Scan
on: [push, pull_request]

jobs:
  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # ต้องมี history ทั้งหมด
      
      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### GitHub's Built-in Secret Scanning

GitHub ตอนนี้มี **secret scanning + push protection** built-in (ฟรีสำหรับ public repos และ public organizations):

- Settings → Code security → Secret scanning → Enable
- Push protection → Enable

→ ถ้ามี secret ใน push, GitHub block ก่อนรับ (เหมือนที่เกิดในบทที่ 6 ตอนเขียน chapter Cryptography)

---

## Secret Rotation — เปลี่ยน Key เป็นประจำ

แม้คุณ store secrets ดีแค่ไหน — secrets ที่ใช้นานๆ มี **risk สะสม**:

### ทำไมต้อง Rotate

1. **Key ที่ใช้นานๆ มีโอกาสรั่ว** — มากเท่าไหร่ที่ใช้ ก็มีโอกาสเห็นโดยคนไม่ควรเห็นมากขึ้น
2. **พนักงานออก** — ถ้า key เก่ายังใช้ได้ = อดีตพนักงานเข้าถึงได้
3. **Compliance** — PCI-DSS, SOC 2, HIPAA require rotation
4. **Damage Control** — ถ้า key หลุด rotation ทำให้ key เก่าหมดอายุเอง

### Rotation Schedule ที่แนะนำ

| Secret Type | Rotation Frequency |
|---|---|
| Database root passwords | Quarterly |
| API keys (3rd party) | 90 days |
| OAuth client secrets | 6-12 months |
| TLS certificates | 90 days (Let's Encrypt) — auto-renew |
| Service account keys | 30-90 days |
| IAM access keys | 90 days |
| JWT signing keys | Yearly (with grace period) |
| Encryption keys | Yearly (with re-encryption process) |

### Automatic Rotation

#### AWS Secrets Manager

```bash
# Setup automatic rotation สำหรับ RDS
aws secretsmanager rotate-secret \
    --secret-id prod/db/credentials \
    --rotation-lambda-arn arn:aws:lambda:... \
    --rotation-rules AutomaticallyAfterDays=30
```

→ AWS จะ rotate password ของ RDS อัตโนมัติทุก 30 วัน

#### HashiCorp Vault Dynamic Secrets

Vault สามารถสร้าง **temporary credentials** ที่หมดอายุเอง:

```bash
# ขอ database credentials ที่ valid 1 ชั่วโมง
vault read database/creds/my-role
# → username: v-token-my-role-xxx
#   password: xxx
#   lease_duration: 3600
```

App ขอใหม่ก่อนหมดอายุ → ไม่ต้อง rotate manually

#### IAM Access Keys

```bash
# AWS CLI
aws iam create-access-key --user-name myuser
# Update apps to use new key
aws iam delete-access-key --access-key-id AKIA... --user-name myuser
```

ใน CI/CD pipeline สามารถ automate ได้

### Rotation Best Practices

1. **Two-key strategy** — มี active + standby ทุกเวลา
2. **Grace period** — old key ยัง valid 24-72 ชั่วโมงหลัง rotate
3. **Monitor usage** — ตรวจว่า apps ทุกตัวเปลี่ยนเป็น new key แล้ว
4. **Audit log** — log ทุกครั้งที่ rotate เกิดขึ้น
5. **Test rotation** — ก่อน production ลองใน staging ก่อน

---

## ถ้า Secret หลุดแล้ว — Incident Response

### Step 1: Confirm รั่วจริง (5 นาทีแรก)

- เปิด GitHub repo / leak source ดูว่า key อะไร
- Note timestamp ที่ key อยู่ใน public

### Step 2: Revoke ทันที

#### AWS

```bash
# Disable IAM access key ทันที
aws iam update-access-key --access-key-id AKIA... --status Inactive --user-name myuser

# Delete หลัง confirm ไม่มี service ที่ใช้แล้ว
aws iam delete-access-key --access-key-id AKIA... --user-name myuser
```

#### Stripe

```
Dashboard → Developers → API Keys → Roll/Reveal → "Reveal Live Key"
→ "Roll Key" (สร้างใหม่ + invalidate เก่า)
```

#### Database

```sql
-- Change password ทันที
ALTER USER db_user WITH PASSWORD '<new-strong-password>';
-- + Force disconnect existing sessions
SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE usename = 'db_user';
```

### Step 3: ประเมิน Damage

- Check audit logs (CloudTrail, GCP audit log) — ดูว่ามีใครใช้ key นี้ไปทำอะไร
- Monitor billing — มี charge ผิดปกติไหม
- Check resource creation — มี EC2/S3/etc. ที่ไม่ใช่ของเราไหม

### Step 4: Cleanup

- Delete unauthorized resources
- Revoke other secrets ที่อาจรั่วด้วย
- Reset passwords สำหรับ user account ที่เกี่ยวข้อง

### Step 5: Document + Notify

- Internal incident report
- ถ้ารั่วในขนาดที่กระทบ user → notification ตามกฎหมาย (PDPA / GDPR)
- ถ้าเป็น customer data → notify customers

### Step 6: Prevent

- Setup pre-commit hooks
- Setup CI scan
- Update onboarding docs
- Post-mortem with team

### Common Mistake: Rotation ไม่พอ

หลายคนคิดว่า "rotate key เก่า → จบ"

**ผิด** — ต้องเช็คว่าคนที่ได้ key เก่าไป **ทำอะไรไปแล้ว**:

- สร้าง backdoor account
- เปลี่ยน security settings
- Exfiltrate data ไป
- ตั้ง webhook ที่ส่งข้อมูลออก

→ ต้อง **forensic investigation** เต็มรูป ไม่ใช่แค่ rotate

---

## Hands-on: Setup Secret Management ใน Project

### สำหรับ Solo Dev / Small Team

#### Step 1: สร้าง `.env` Strategy

```bash
# ใน root ของ project
echo "*.env\n*.env.local" >> .gitignore

cat > .env.example << EOF
# Database
DATABASE_URL=

# 3rd party APIs
STRIPE_SECRET_KEY=
SENDGRID_API_KEY=

# JWT
JWT_SECRET=
EOF

git add .gitignore .env.example
git commit -m "Setup env strategy"
```

#### Step 2: ติดตั้ง Pre-commit Hooks

```bash
pip install pre-commit
cat > .pre-commit-config.yaml << EOF
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
EOF

pre-commit install
git add .pre-commit-config.yaml
git commit -m "Add pre-commit hook for secret scanning"
```

#### Step 3: เลือก Secret Manager

แนะนำเริ่มจาก:
- **Doppler** — ฟรี tier เพียงพอ, DX ดี
- หรือ **AWS Secrets Manager** ถ้าใช้ AWS อยู่แล้ว

#### Step 4: Migrate Secrets

```bash
# Doppler example
doppler login
doppler setup     # เลือก project + environment
doppler secrets set DATABASE_URL="<your-db-url>"
doppler secrets set STRIPE_SECRET_KEY="<your-stripe-key>"

# Run app
doppler run -- npm start
```

#### Step 5: Configure CI/CD

```yaml
# GitHub Actions
- name: Setup Doppler
  uses: dopplerhq/cli-action@v3

- name: Run tests
  run: doppler run --token=${{ secrets.DOPPLER_TOKEN }} -- npm test
```

### สำหรับ Mid-size / Enterprise

#### 1. Adopt Vault หรือ Cloud-native

Setup HashiCorp Vault หรือใช้ AWS/GCP Secret Manager + IAM-based access

#### 2. RBAC Policy

```hcl
# Vault policy
path "secret/data/myapp/*" {
  capabilities = ["read"]
}

path "secret/data/myapp/admin/*" {
  capabilities = ["read", "create", "update"]
}
```

แต่ละ team / service ได้แค่ scope ที่ต้องใช้

#### 3. Audit Logging

- ทุก access ของ secret → log
- Alert ถ้ามี access pattern ผิดปกติ

#### 4. Automated Rotation

- DB password rotate auto
- API key rotation pipeline
- Cert renewal automation

#### 5. Disaster Recovery

- Vault HA cluster
- Backup ของ vault data
- DR site

---

## Action Items

### วันนี้ (ทุก Developer)

- [ ] **Audit project ปัจจุบัน** — `grep -ri "api_key\|password\|secret" --exclude-dir={node_modules,.git}` ดูว่ามี hardcoded secret ไหม
- [ ] **เพิ่ม `.env`, `.env.local` ใน `.gitignore`**
- [ ] **ติดตั้ง gitleaks หรือ trufflehog** + ลอง scan repo เก่า

### สัปดาห์นี้

- [ ] **เลือก secret manager** (Doppler / AWS Secrets Manager / Vault)
- [ ] **Migrate secrets** ออกจาก `.env` files ใน production → secret manager
- [ ] **Setup pre-commit hooks** — gitleaks, detect-secrets
- [ ] **Enable GitHub secret scanning + push protection** ใน org

### เดือนนี้

- [ ] **Audit history ทั้งหมด** — `gitleaks detect --source . --log-opts="--all"`
- [ ] **Rotate ทุก secret** ที่เคยอยู่ใน Git history
- [ ] **Document secret management policy**
- [ ] **Train ทีม** เรื่อง secret management

### สำหรับ Tech Lead

- [ ] **Centralized secret manager** สำหรับทั้ง org
- [ ] **Rotation policy** + automated rotation ที่ทำได้
- [ ] **Audit log + alerting** สำหรับ secret access ผิดปกติ
- [ ] **Onboarding playbook** ที่บอก dev ใหม่
- [ ] **Incident response plan** สำหรับ secret leak

### สำหรับ DevSecOps

- [ ] **CI/CD integration** — secret scan ใน pipeline
- [ ] **Block PR ที่มี secret** — automated
- [ ] **Quarterly review** — ตรวจ access ที่อนุญาต
- [ ] **Threat modeling** — แต่ละ secret มีสิทธิ์อะไร และใครเข้าถึงได้

---

## บทเรียนชีวิตจากบทความนี้

> **อย่าเก็บของมีค่าไว้ในที่ที่คนทั่วไปเข้าถึงได้**

ดูเหมือนเป็นเรื่องที่ "เห็นชัดอยู่แล้ว" — แต่ในชีวิตจริงคนทำผิดบ่อยมาก:

- **เก็บกุญแจสำรอง ใต้พรมหน้าบ้าน** — โจรรู้
- **เก็บรหัสผ่าน ATM ใน wallet** — wallet หาย = หมด
- **เก็บเลขบัญชี + PIN ในมือถือไม่มี passcode**
- **เก็บเอกสารสำคัญ ในกล่องใต้เตียง** — ไฟไหม้ = หมด
- **เปิดเผยเงินเดือน + bonus** ในที่ทำงาน — ไม่จำเป็น

ในยุคดิจิทัล เพิ่มอีกระดับ:

- **โพสต์รูปบัตรเครดิต** ลง Instagram (มีคนทำจริง!)
- **Share รหัสผ่านในแชท Line / Slack**
- **เก็บเอกสาร tax ใน Google Drive ที่ share เป็น "anyone with link"**
- **บอก security question ของบัญชีในการแชท**

> **กฎทอง: ก่อนเก็บอะไรที่ไหน — ถามตัวเองว่า "ถ้าคนที่ไม่ควรเห็น เห็น อันตรายแค่ไหน?"**

ถ้าอันตราย → หาที่ปลอดภัยกว่า  
ถ้าไม่อันตราย → สบายใจ

หลักการเดียวกันใช้ในการแยก **ระดับความเชื่อใจ**:

#### ระดับ 1: สาธารณะ — ใครเห็นก็ได้
- ชื่อ, หน้าตา (ในการประชุมตัวต่อตัว)
- ข้อมูลบริษัทที่ public

#### ระดับ 2: ใกล้ตัว — เพื่อนร่วมงาน, เพื่อน
- เบอร์โทร
- Email work
- ตำแหน่งงาน

#### ระดับ 3: ครอบครัว / Inner Circle
- เงินเดือน
- ปัญหาส่วนตัว
- บัญชีธนาคาร

#### ระดับ 4: ตัวเองเท่านั้น
- รหัสผ่าน
- Recovery codes
- พินัยกรรม
- เอกสาร confidential

> **คนที่ "vault ของชีวิต" ดี — อยู่รอดในระยะยาวง่ายกว่า**

ในเรื่อง code ก็เช่นกัน:

- **ระดับ 1: source code logic** → public ได้ (open source)
- **ระดับ 2: API endpoints** → documented ได้
- **ระดับ 3: configurations** → internal เท่านั้น
- **ระดับ 4: credentials** → ใน vault เท่านั้น เข้าถึงได้เฉพาะที่จำเป็น

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไป เราจะเข้าเรื่อง **Supply Chain Attack** — ภัยที่ผ่าน "สิ่งที่คุณ trust" — เช่น library, dependency, vendor — และเป็นเรื่องที่ dev ทุกคนต้องระวังในยุคนี้

---

## อภิธานศัพท์ (Glossary)

- **Secret:** ข้อมูลที่ต้องเก็บเป็นความลับ — password, API key, certificate, encryption key
- **Hardcoded Secret:** secret ที่อยู่ใน source code ตรงๆ (ห้ามทำ)
- **Environment Variable:** ตัวแปรที่ระบบส่งให้ application (เช่น `process.env.X`)
- **`.env` File:** ไฟล์ที่เก็บ env variables สำหรับ local dev
- **Secret Manager:** บริการที่เก็บ + จัดการ secrets แบบ centralized
- **Vault:** เครื่องมือเก็บ secrets ที่ encrypted + access-controlled
- **Static Secret:** secret ที่ไม่เปลี่ยนเอง — ต้อง rotate manually
- **Dynamic Secret:** secret ที่ generate on-demand + expire automatically (Vault feature)
- **Secret Rotation:** การเปลี่ยน secret เป็นระยะ
- **Pre-commit Hook:** script ที่รันก่อน `git commit` — ใช้ scan secrets ฯลฯ
- **Gitleaks / TruffleHog:** เครื่องมือ scan repo หา secrets
- **Sealed Secrets:** Kubernetes secrets ที่ encrypt แล้ว commit ลง Git ได้
- **SOPS:** เครื่องมือ encrypt YAML/JSON ด้วย cloud KMS
- **External Secrets Operator:** k8s tool ที่ sync secrets จาก external manager
- **HSM (Hardware Security Module):** อุปกรณ์ hardware ที่เก็บ key ปลอดภัยสุด
- **Audit Log:** บันทึกทุกการเข้าถึง secrets
- **Principle of Least Privilege:** ให้สิทธิ์เท่าที่จำเป็นเท่านั้น

---

## สรุป

1. **Hardcoded secrets ใน code = ภัยอันดับหนึ่ง** — รั่วใน 38 วินาที average
2. **Private repo ไม่ได้แปลว่าปลอดภัย** — visibility อาจเปลี่ยน, collaborator อาจรั่ว
3. **ใช้ environment variables + `.env` + `.gitignore`** เป็นพื้นฐาน
4. **Production ต้องการ secret manager** — Doppler, AWS Secrets Manager, Vault
5. **Pre-commit hooks** กันก่อนหลุด — gitleaks, trufflehog
6. **Rotate secrets เป็นระยะ** — automated เมื่อทำได้
7. **ถ้า secret หลุด** — revoke ทันที, audit damage, forensic investigation
8. **Verify ทุกอย่าง** — CI scan, audit log, alert บน access ผิดปกติ
9. **Layered approach** — แต่ละ secret มี scope + sensitivity ต่างกัน

ในบทถัดไป **Supply Chain Attack** — เราจะมาดูว่าแม้ secrets ของคุณปลอดภัย dependency ที่คุณใช้อาจกลับมาทำร้ายคุณยังไง

— Claude Opus 4.6
