# บทความที่ 16: Cloud Security — เมื่อ Infrastructure อยู่บน Cloud ความเสี่ยงก็ตามไปด้วย

**CyberSecurity Awareness Series — Part 5: Developer-Specific**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของ Capital One — เมื่อ Misconfigured Cloud = ข้อมูล 100 ล้านคนหลุด

เดือนกรกฎาคม ปี 2019 — **Paige Thompson** อดีตพนักงาน AWS โพสต์ใน Slack ส่วนตัว:

> *"I have a leaked database from a US bank — 30 GB customer data."*

โพสต์ถูก screenshot ส่งให้ Capital One — ภายใน 2 สัปดาห์ FBI จับกุม Paige และเปิดเผย:

**Capital One** บริษัทธนาคาร US ตัวใหญ่ ถูกแฮกข้อมูลของลูกค้า **มากกว่า 100 ล้านคน** (ประมาณ 1 ใน 3 ของประชากรอเมริกา):

- ชื่อ, address, date of birth, email
- Credit score, balance, transaction history
- **140,000 SSN** + 80,000 bank account numbers (US)
- 1 ล้าน Canadian Social Insurance Numbers

วิธีที่เกิดเหตุ — **ไม่ใช่ phishing, ไม่ใช่ malware, ไม่ใช่ ransomware** — เป็น **misconfigured AWS** ตรงๆ:

1. **Web Application Firewall (WAF)** ของ Capital One มีช่องโหว่ **SSRF**
2. WAF run บน EC2 ที่มี IAM role ที่มี **S3 read access**
3. Paige ใช้ SSRF เพื่อขอ **EC2 metadata service** (`169.254.169.254`)
4. ได้ AWS credentials ของ IAM role
5. ใช้ credentials นั้น list + read S3 buckets — ที่มีข้อมูล customer

ความเสียหาย:

- **$80 million** ค่าปรับจาก US OCC (Office of the Comptroller of the Currency)
- **$190 million** จาก class action settlement
- Reputation damage ตีค่าไม่ได้
- **$0** ของจริงที่ AWS ถูก fault — Capital One ผิด config เอง

[อ่านรายละเอียด Capital One breach](https://en.wikipedia.org/wiki/2019_Capital_One_data_breach)

ที่น่าตกใจ — Paige ไม่ได้เป็น hacker ระดับสูงหรือ state-sponsored attacker เธอเป็น **อดีตพนักงาน AWS** ที่รู้ว่า config แบบไหนผิด แล้วลองในเว็บคนอื่น

> **บนคลาวด์ — ไม่ใช่ "hack ยาก" แต่ "config ถูกต้อง" ยาก**

นี่คือเหตุผลว่าทำไม cloud security คือเรื่องที่ทุก dev ที่ deploy บน cloud ต้องเข้าใจ — และทำไม **misconfiguration เป็นสาเหตุที่ใหญ่ที่สุด** ของ cloud breach ทุกปี

---

## Shared Responsibility Model — รู้ก่อนว่าใครรับผิดชอบอะไร

ก่อนเริ่ม — ต้องเข้าใจ **Shared Responsibility Model** ที่ AWS / GCP / Azure ใช้กันทั้งหมด:

> **Cloud provider รับผิดชอบ "security OF the cloud"**  
> **คุณรับผิดชอบ "security IN the cloud"**

### AWS Shared Responsibility Model

```
┌──────────────────────────────────────────────────┐
│ ลูกค้า (You) รับผิดชอบ:                         │
│  - Customer Data                                 │
│  - Platform, Application, Identity Management   │
│  - Operating System, Network, Firewall config   │
│  - Client-side data encryption                  │
│  - Server-side encryption (file, DB, etc.)      │
│  - Network traffic protection (TLS, etc.)       │
└──────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────┐
│ AWS รับผิดชอบ:                                   │
│  - Software:                                     │
│    Compute, Storage, Database, Networking       │
│  - Hardware:                                     │
│    Regions, Availability Zones, Edge Locations  │
└──────────────────────────────────────────────────┘
```

### หลักการง่ายๆ

| Service Type | AWS รับผิดชอบ | คุณรับผิดชอบ |
|---|---|---|
| **IaaS** (EC2, EBS) | Hardware, hypervisor | OS, app, data, network config, IAM |
| **PaaS** (RDS, Elastic Beanstalk) | + OS patching, runtime | Data, schema, IAM, app code |
| **SaaS** (S3, Lambda, DynamoDB) | + Application | Data, IAM, configuration |

### ตัวอย่างการแยก responsibility

#### EC2 (IaaS)

- AWS ดูแล: physical server, hypervisor, network hardware
- คุณดูแล: OS updates, app, security groups, IAM, encryption

#### S3 (SaaS-like Storage)

- AWS ดูแล: storage hardware, S3 service, encryption infrastructure
- คุณดูแล: bucket policy, who can access, encryption settings, versioning

#### RDS (PaaS DB)

- AWS ดูแล: hardware, OS, database engine patches
- คุณดูแล: schema, queries, IAM, encryption choices, network isolation

### ทำไมเรื่องนี้สำคัญ

หลายคนคิดว่า:

> "ใช้ AWS แล้วปลอดภัย"

**ผิด** — AWS ทำให้ "infrastructure" ปลอดภัย แต่ **คุณ** ต้อง configure ให้ถูก

99% ของ cloud data breach ในปี 2026 — เกิดจาก **customer misconfiguration** ไม่ใช่ provider failure

### ตัวอย่างความแตกต่าง AWS vs GCP vs Azure

#### AWS

- IAM ละเอียดที่สุด (roles, policies, conditions)
- Service เยอะที่สุด (~200+)
- Marketplace ใหญ่
- Documentation เยอะ + complex

#### GCP

- IAM แตกต่าง (resource hierarchy: Org → Folder → Project)
- Cleaner UX กว่า AWS
- Strong on data + AI services
- Identity-Aware Proxy (IAP)

#### Azure

- IAM ผ่าน Microsoft Entra ID (เดิม Azure AD)
- Tightly integrated กับ Microsoft ecosystem
- เน้น enterprise / hybrid cloud

หลักการ security เหมือนกัน — แต่ tooling + naming ต่างกัน

---

## IAM — หัวใจของ Cloud Security

**IAM** (Identity and Access Management) = ระบบที่ตอบ:
- **Who?** (identity)
- **Can do what?** (permissions)
- **On what?** (resources)
- **Under what conditions?** (e.g., from certain IPs, with MFA, etc.)

### Identity Types

#### Users (Human)

```
Tanin Thanasopon (tanin@company.com)
   ↓
   - AWS IAM User
   - Member of group: Developers
   - Has access keys (for CLI)
   - Has password (for console)
   - MFA enabled
```

#### Service Accounts / IAM Roles (Machine)

```
EC2 instance running web app
   ↓
   - Assumed Role: webapp-prod-role
   - Permissions: read S3 bucket "user-uploads"
   - No long-term credentials!
   - Credentials rotated automatically every few hours
```

### Principle of Least Privilege สำหรับ IAM

> **Default to deny. Grant explicit permissions only.**

#### Bad: Wildcards Everywhere

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "*",          // ❌ ทำได้ทุกอย่าง
    "Resource": "*"         // ❌ ทุก resource
  }]
}
```

This is essentially **AdministratorAccess** — ใช้กับ user / role ในงานประจำวัน = อันตราย

#### Good: Specific Permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ],
    "Resource": "arn:aws:s3:::user-uploads-bucket/*",
    "Condition": {
      "IpAddress": {
        "aws:SourceIp": "10.0.0.0/8"
      }
    }
  }]
}
```

→ Only `GetObject` + `PutObject` บน specific bucket จาก specific IP

### Access Patterns

#### Don't Use Root Account

AWS root account = ลูกกุญแจหลักของบ้าน
- ใช้แค่ตอน setup ครั้งแรก
- เปิด **MFA**
- เก็บ credentials ใน safe — ไม่ใช้ปกติ
- ไม่สร้าง access keys สำหรับ root

#### Don't Use Long-lived Access Keys (ถ้าหลีกเลี่ยงได้)

```bash
# ❌ Long-lived access key
[default]
aws_access_key_id = <your-access-key>
aws_secret_access_key = <your-secret>

# Risk: ถ้า leak = เปิดให้ใช้ตลอดไป จนกว่า rotate
```

```bash
# ✅ Short-lived credentials via SSO / SAML / IAM Identity Center
aws sso login
# Get temp credentials valid for 8 hours
```

หรือใช้:
- **IAM Identity Center** (AWS SSO) — for users
- **IAM Roles for Service Accounts** (IRSA) — for k8s pods
- **EC2 Instance Profile** — for EC2 instances
- **OIDC federation** — for GitHub Actions / GitLab CI

#### Use IAM Roles, Not Users สำหรับ Workloads

```
[GitHub Actions] → OIDC → AWS IAM Role
                      ↓
                   Temp credentials (1 hour)
                      ↓
                   Deploy
```

ไม่ต้องเก็บ AWS access key ใน GitHub Secrets

### Common IAM Mistakes

#### 1. Over-permissive Policies

```json
// ❌
"Action": "s3:*"  // includes Delete, Replication, etc.

// ✅
"Action": ["s3:GetObject", "s3:PutObject", "s3:ListBucket"]
```

#### 2. Resource Wildcards When Specific Should Work

```json
// ❌
"Resource": "*"  // ทุก S3 bucket ในบัญชี

// ✅
"Resource": "arn:aws:s3:::my-app-bucket/*"
```

#### 3. Trust Policy Too Open

```json
// ❌ AssumeRole จาก *
{
  "Effect": "Allow",
  "Principal": "*",
  "Action": "sts:AssumeRole"
}

// ✅ Specific principal
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::123456789012:role/lambda-role"
  },
  "Action": "sts:AssumeRole"
}
```

#### 4. ลืม Conditions

Conditions = restrict ตาม context:

```json
{
  "Condition": {
    "Bool": {
      "aws:MultiFactorAuthPresent": "true"
    },
    "IpAddress": {
      "aws:SourceIp": ["10.0.0.0/8", "172.16.0.0/12"]
    },
    "DateGreaterThan": {
      "aws:CurrentTime": "2026-01-01T00:00:00Z"
    }
  }
}
```

#### 5. Inline Policies vs Managed Policies

- **Inline Policy** = แนบกับ user/role โดยตรง — review ยาก
- **Managed Policy** = standalone, attach กับหลายตัว — review + audit ง่าย

→ ใช้ Managed Policies

### IAM Best Practices Checklist

- [ ] **MFA** บน root + ทุก user
- [ ] **Root account** ไม่มี access keys
- [ ] **IAM Identity Center** หรือ SSO สำหรับ user access
- [ ] **No long-lived access keys** สำหรับ workload
- [ ] **Roles for services** ไม่ใช่ users
- [ ] **Least privilege** policies (no `*:*`)
- [ ] **Permission boundaries** สำหรับ user-created roles
- [ ] **Service control policies** (SCP) ใน Organizations
- [ ] **IAM Access Analyzer** เปิดใช้งาน
- [ ] **Access keys rotation** every 90 days (if used)
- [ ] **Audit IAM permissions** เป็นระยะ

---

## ภัยที่พบบ่อย: Cloud Misconfigurations

### Misconfiguration 1: S3 Bucket Public

นี่คือ misconfiguration ที่พบบ่อยที่สุดในประวัติศาสตร์ของ cloud:

**กรณีจริง:**

- **Booz Allen Hamilton (2017)** — geospatial intelligence data ของกองทัพอเมริกาใน public S3
- **Verizon (2017)** — 14 ล้าน customer records
- **Accenture (2017)** — security keys + auth credentials
- **Tesla (2018)** — Kubernetes console เปิด public + S3 leak
- **Capital One (2019)** — ที่กล่าวข้างต้น
- **TikTok (2024)** — รายงานว่าเปิด public โดยไม่ตั้งใจ

#### Causes

```python
# ❌ Granting "AllUsers" or "AuthenticatedUsers"
s3.put_bucket_acl(
    Bucket='my-bucket',
    ACL='public-read'
)

# ❌ Bucket policy ที่อนุญาต *
{
  "Effect": "Allow",
  "Principal": "*",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

#### Defense

**1. Block Public Access (Account-level)**

```bash
aws s3control put-public-access-block \
    --account-id 123456789012 \
    --public-access-block-configuration \
        BlockPublicAcls=true,\
        IgnorePublicAcls=true,\
        BlockPublicPolicy=true,\
        RestrictPublicBuckets=true
```

→ Block ทุก public access ในระดับ account — ใช้ exception เป็นรายๆ

**2. Bucket-level Block Public Access**

ตั้งสำหรับ bucket แต่ละตัวด้วย

**3. AWS Macie / GuardDuty**

Detect sensitive data ใน public buckets อัตโนมัติ

**4. Continuous Monitoring**

```bash
# Tools ที่ scan automatically
- AWS Config Rules (s3-bucket-public-read-prohibited)
- Prowler — open source CSPM
- Steampipe — SQL queries บน cloud config
```

### Misconfiguration 2: Security Groups เปิดกว้าง

**Security Groups** = firewall rules สำหรับ EC2

#### Bad Examples

```bash
# ❌ Database port เปิดทั่วโลก
Inbound:
  Port 5432 (PostgreSQL): 0.0.0.0/0   # 💥
  Port 27017 (MongoDB): 0.0.0.0/0     # 💥
  Port 6379 (Redis): 0.0.0.0/0        # 💥

# ❌ SSH เปิดทั่วโลก
Inbound:
  Port 22 (SSH): 0.0.0.0/0            # 💥 brute force ได้
```

#### Good Examples

```bash
# ✅ Database — เฉพาะ app server
Inbound:
  Port 5432: sg-app-server (security group ID)

# ✅ SSH — เฉพาะ bastion host
Inbound:
  Port 22: sg-bastion-host

# ✅ HTTPS — public web
Inbound:
  Port 443: 0.0.0.0/0   # OK สำหรับ public website
```

**กรณีจริง:**

- **MongoDB databases** เปิด public + ไม่มี auth = data รั่ว >100 ล้าน records ใน 2017-2024
- **Redis** เปิด public = used for cryptojacking
- **Elasticsearch** เปิด public = data indexing visible

#### Tools

- **AWS Trusted Advisor** — alerts สำหรับ open security groups
- **GuardDuty** — detect suspicious activity
- **VPC Flow Logs** — log network traffic + analyze

### Misconfiguration 3: Database Exposed

#### Bad

```yaml
# ❌ RDS public + open security group
RDS PostgreSQL:
  Publicly Accessible: true
  Security Group: db-sg
    Inbound: 0.0.0.0/0:5432

# Result: RDS reachable จาก internet → brute force
```

#### Good

```yaml
# ✅ RDS in private subnet, app subnet only
RDS PostgreSQL:
  Publicly Accessible: false
  VPC: my-vpc
  Subnet Group: private-db-subnets
  Security Group:
    Inbound: sg-app-tier:5432
```

ทำให้ DB เข้าได้แค่จาก app server ใน VPC เดียวกัน

### Misconfiguration 4: IAM Excessive Permissions

ที่กล่าวข้างต้น — root access keys, wildcard permissions, no MFA

### Misconfiguration 5: Logging Disabled

```yaml
# ❌
CloudTrail: disabled
S3 Server Access Logs: disabled
VPC Flow Logs: disabled

# ↓ กรณีโดน hack — ไม่มี evidence ว่าเกิดอะไร
```

```yaml
# ✅
CloudTrail: enabled (multi-region, log file integrity validation)
S3 Server Access Logs: enabled
VPC Flow Logs: enabled
GuardDuty: enabled
```

### Misconfiguration 6: Encryption Disabled

```yaml
# ❌
S3 Bucket: SSE disabled
RDS: encryption at rest disabled
EBS Volume: encryption disabled

# ↑ Default ใน AWS ปัจจุบันคือ "encrypted" — แต่บาง config เก่าอาจยังไม่ใช่
```

```yaml
# ✅
S3 Bucket: SSE-KMS enabled
RDS: encryption with customer-managed KMS key
EBS Volume: encryption enabled (default)
```

### Misconfiguration 7: ENI / VPC Peering Loose

VPC peering ที่ไม่จำกัด traffic = ขยาย attack surface

---

## Infrastructure as Code (IaC) Security

ที่กล่าวในบทที่ 11 — IaC ดี เพราะ reproducible + version controlled

แต่ IaC ก็เป็น **point of attack** ได้:

### IaC Risks

1. **Misconfiguration ใน Terraform** → deploy ไป production = ผิดทุก environment
2. **Hardcoded secrets** ใน .tf files
3. **State file** มี sensitive data
4. **Public modules** ที่อาจเป็น malicious

### Scan IaC ก่อน Deploy

#### Checkov

[checkov.io](https://www.checkov.io) — open source

```bash
pip install checkov

# Scan Terraform
checkov -d ./terraform/

# ตัวอย่าง output:
# Check: CKV_AWS_18: "Ensure the S3 bucket has access logging enabled"
# 	FAILED for resource: aws_s3_bucket.my_bucket
# 	File: /terraform/main.tf:5-12
```

#### tfsec

```bash
brew install tfsec
tfsec ./terraform/

# Detect:
# - Public S3 buckets
# - Open security groups
# - Unencrypted resources
# - Missing encryption keys
```

#### Bridgecrew (Prisma Cloud)

Commercial — integrate กับ GitHub PR

#### Terrascan

Multi-IaC support (Terraform, k8s, Helm, AWS CFN)

### CI Integration

```yaml
# .github/workflows/iac-scan.yml
name: IaC Security Scan
on: [pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/
          quiet: true
          soft_fail: false  # fail PR if violations
      
      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1.0.0
```

### Terraform State Security

```yaml
# ✅ Remote state ใน S3 + DynamoDB locking
terraform {
  backend "s3" {
    bucket         = "company-tf-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true                          # ✅ encrypted
    dynamodb_table = "tf-state-lock"               # ✅ lock
    kms_key_id     = "arn:aws:kms:us-east-1:..."  # ✅ KMS key
  }
}
```

→ State file encrypted + locked + versioned

---

## Logging & Monitoring บน Cloud

### AWS

#### CloudTrail — Track API Calls

```yaml
# ตั้ง trail ที่ครอบคลุม
- Multi-region: yes
- Management events: read + write
- Data events: S3, Lambda
- Insights events: yes
- Log file integrity validation: yes
- Encryption: KMS
- Retention: 1+ year
```

#### CloudWatch Logs

```yaml
- Log group สำหรับ application + infrastructure
- Subscription filter ส่งไป SIEM
- Metric filters + alarms สำหรับ suspicious patterns
```

#### GuardDuty

```yaml
- Threat detection สำหรับ:
  - Compromised EC2
  - Cryptojacking
  - Reconnaissance from suspicious IPs
  - Anomalous API calls
```

#### AWS Config

```yaml
- Track configuration changes
- Compliance rules (CIS, PCI-DSS, etc.)
- Auto-remediation for non-compliant resources
```

#### Security Hub

```yaml
- Unified view ของ security findings
- รวม GuardDuty + Inspector + Macie + Config
- Compliance dashboards
```

### Critical Alerts ที่ควรตั้ง

```yaml
- Root account login
- IAM user / role created or deleted
- Security group changes
- VPC config changes
- S3 bucket policy changes
- KMS key disabled
- CloudTrail disabled (!!!)
- Failed login attempts (>5)
- API calls from unusual region
- Large data transfer out
- New EC2 instances in unexpected regions
- Cryptocurrency mining patterns
```

### GCP Equivalent

- **Cloud Audit Logs** = CloudTrail
- **Cloud Logging** = CloudWatch Logs
- **Security Command Center** = Security Hub + GuardDuty
- **Cloud Asset Inventory** = AWS Config

### Azure Equivalent

- **Activity Log** = CloudTrail
- **Azure Monitor** = CloudWatch
- **Microsoft Defender for Cloud** = Security Hub + GuardDuty

### SIEM Integration

ส่ง logs ทั้งหมดไป SIEM (Security Information and Event Management):

- **Splunk**
- **Elastic Security**
- **Datadog Security**
- **Sumo Logic**
- **Microsoft Sentinel**

→ Correlate events + detect attacks ที่กระจายในหลาย service

---

## Container Security Basics

ในยุคปี 2026 — ทุก app deploy ผ่าน container (Docker / Kubernetes)

### Container Image Security

#### Use Minimal Base Images

```dockerfile
# ❌ Bloated
FROM ubuntu:latest          # ~70MB, มี shell, package manager, ฯลฯ
RUN apt-get update && apt-get install -y python3
COPY app.py /app
CMD ["python3", "/app/app.py"]

# ✅ Distroless
FROM gcr.io/distroless/python3
COPY app.py /app
CMD ["/app/app.py"]
# ~50MB ไม่มี shell, package manager → attack surface ลด

# ✅ Or Alpine
FROM python:3.11-alpine
# ~50MB, minimal Linux
```

ยิ่งเล็ก ยิ่งปลอดภัย:
- Vulnerability surface ลด
- Less software ที่ต้อง patch
- Image transfer เร็ว

#### Don't Run as Root

```dockerfile
# ❌
FROM python:3.11
COPY app.py /app
CMD ["python3", "/app/app.py"]
# Default: root user → ถ้า exploit = root access

# ✅
FROM python:3.11
RUN useradd -m -u 1000 appuser
USER appuser
COPY --chown=appuser:appuser app.py /app
CMD ["python3", "/app/app.py"]
```

#### Don't Hardcode Secrets

```dockerfile
# ❌
ENV API_KEY="<your-api-key>"

# ✅
# Pass at runtime
docker run -e API_KEY=$API_KEY my-app
# Or via secret manager
```

#### Pin Versions

```dockerfile
# ❌ Latest tag → ไม่ reproducible + อาจมี malware ใน update
FROM node:latest

# ✅ Specific version + sha256 digest
FROM node:18.18.0@sha256:abc123...
```

#### Multi-stage Builds

```dockerfile
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
CMD ["node", "dist/index.js"]
```

→ Final image ไม่มี build tools / source code

### Image Scanning

```bash
# Trivy
trivy image my-app:latest

# Snyk
snyk container test my-app:latest

# Grype
grype my-app:latest

# Docker Scout
docker scout cves my-app:latest
```

Integrate ใน CI/CD — block deploy ถ้ามี critical vulnerabilities

### Kubernetes Security

#### Pod Security

```yaml
apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: app
      image: my-app:1.0
      securityContext:
        readOnlyRootFilesystem: true
        allowPrivilegeEscalation: false
        capabilities:
          drop: ["ALL"]
      resources:
        limits:
          cpu: 500m
          memory: 512Mi
```

#### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-by-default
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

→ Default deny — explicit allow

#### RBAC

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: app-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

#### Tools

- **kube-bench** — CIS Kubernetes Benchmark check
- **Falco** — runtime security monitoring
- **OPA Gatekeeper** — policy enforcement
- **Polaris** — best practice checking

---

## Hands-on: Audit IAM Permissions ของทีมคุณวันนี้

### Step 1: List ทุก IAM User + Role

```bash
# List users
aws iam list-users

# List roles  
aws iam list-roles

# List access keys (per user)
aws iam list-access-keys --user-name <name>
```

### Step 2: Identify Risky Configurations

```bash
# Users with admin access
aws iam list-attached-user-policies --user-name <name>

# Look for: AdministratorAccess, PowerUserAccess

# Old access keys
aws iam list-access-keys --user-name <name> | jq '.AccessKeyMetadata[] | select(.CreateDate < "2025-01-01")'
```

### Step 3: ใช้ IAM Access Analyzer

```bash
aws accessanalyzer list-analyzers
aws accessanalyzer list-findings --analyzer-arn <analyzer-arn>
```

→ Show external access, unused access, etc.

### Step 4: Review with Tools

#### Prowler

```bash
brew install prowler
prowler aws

# Output: 200+ checks สำหรับ AWS security best practices
```

#### Steampipe

```bash
brew install steampipe
steampipe plugin install aws

# Query like SQL
steampipe query "select user_name, password_last_used 
                 from aws_iam_user 
                 where password_last_used < now() - interval '90 days'"
```

#### Cloudsploit / ScoutSuite

ตัวอื่นๆ ที่ scan ทั้ง config

### Step 5: Remediate

#### Quick wins

```bash
# Disable unused access keys
aws iam update-access-key --access-key-id AKIA... --status Inactive

# Delete unused users (after confirm)
aws iam delete-user --user-name <name>

# Force MFA on console users
# (via IAM policy with condition)
```

#### Long-term

- Migrate to IAM Identity Center / SSO
- Implement permission boundaries
- Setup SCP for org-wide guardrails

---

## Action Items

### วันนี้

- [ ] **Enable MFA** บน root + ทุก IAM user
- [ ] **Block S3 public access** (account-level)
- [ ] **Enable CloudTrail** (multi-region, log file validation)
- [ ] **Run Prowler scan** + review findings

### สัปดาห์นี้

- [ ] **Review IAM policies** — remove `*:*` permissions
- [ ] **Migrate to IAM roles** for workloads (no long-lived keys)
- [ ] **Setup GuardDuty + Security Hub**
- [ ] **Enable encryption at rest** สำหรับ S3, RDS, EBS

### เดือนนี้

- [ ] **IAM Identity Center / SSO** สำหรับ user access
- [ ] **IaC scanning** ใน CI (Checkov / tfsec)
- [ ] **Container image scanning** ใน CI
- [ ] **Network policies** ใน Kubernetes
- [ ] **Quarterly access review** schedule

### สำหรับ Cloud Architect / DevOps Lead

- [ ] **Cloud security framework** (CIS Benchmarks, etc.)
- [ ] **Service Control Policies** ใน AWS Organizations
- [ ] **Centralized logging + SIEM**
- [ ] **Disaster Recovery plan** ที่ test เป็นประจำ
- [ ] **Compliance program** (PCI-DSS, SOC 2, etc. ถ้าจำเป็น)

---

## บทเรียนชีวิตจากบทความนี้

> **"ย้ายบ้าน" ไม่ได้แปลว่าปลอดภัยขึ้นอัตโนมัติ — บ้านใหม่ก็ต้องล็อกประตู ติดกล้อง เหมือนกัน**

หลายคนคิดว่าการ **migrate ไป cloud** = security ดีขึ้นทันที

จริงๆ แล้ว — **ความปลอดภัยขึ้นกับว่าคุณ configure ยังไง** ไม่ใช่ว่า "อยู่บน AWS แล้วปลอดภัย"

หลักการนี้ใช้กับชีวิตจริงทุกเรื่อง:

#### บ้านใหม่ ≠ บ้านปลอดภัย

- บ้านในหมู่บ้านปิดดี — แต่ถ้าคุณไม่ล็อกประตู, คนเข้าได้ง่าย
- กล้องวงจรปิดที่ระบบใหม่ — ถ้าคุณตั้ง password = `1234` ก็ไม่ช่วย
- คอนโดที่มี security 24 ชม. — ถ้าคุณให้รหัสกับคนแปลกหน้าก็ไม่ช่วย

#### งานใหม่ ≠ Career ก้าวหน้า

- เปลี่ยนงานบริษัทใหญ่ — ถ้าคุณยังทำงานเหมือนเดิม = ไม่ก้าวหน้า
- ตำแหน่งที่ดีขึ้น — ถ้าไม่เพิ่ม skill ก็แค่เปลี่ยน label
- เงินเดือนสูงขึ้น — ถ้า lifestyle เพิ่มเร็วกว่า = financially แย่กว่าเดิม

#### ความสัมพันธ์ใหม่ ≠ ปัญหาหาย

- หย่าจาก partner เก่า — ถ้าตัวเองยังเหมือนเดิม → ปัญหาเดิมในความสัมพันธ์ใหม่
- เพื่อนใหม่ — ถ้าตัวเอง toxic → เพื่อนใหม่ก็ห่างไป
- "เริ่มต้นใหม่" — ถ้าไม่เปลี่ยนตัวเอง = แค่เปลี่ยนสถานที่

#### เครื่องมือใหม่ ≠ ผลลัพธ์ดีขึ้น

- ซื้อกล้องราคาแพง — ไม่ได้แปลว่าถ่ายรูปสวยขึ้น
- เปลี่ยน text editor ดีกว่า — ไม่ได้แปลว่าเขียน code ดีขึ้น
- ใช้ AI tool ทันสมัย — ไม่ได้แปลว่าทำงานเก่งขึ้น

> **เครื่องมือ คือ amplifier ของทักษะของคุณ — ทักษะเป็น 0 จะ amplify ก็ได้ 0**

ในเรื่อง cloud:

- **ใช้ AWS** = มีเครื่องมือดี
- **Configure ผิด** = ใช้เครื่องมือดีอย่างไม่ถูก
- **ผลลัพธ์**: รั่วเหมือน on-premises แต่กว้างกว่า (เพราะ cloud reachable from internet)

> **คนที่ฉลาดจริง = ใช้เครื่องมือที่มีอยู่ให้ดีที่สุด ก่อนค่อยซื้อใหม่**  
> **คนที่ไม่ฉลาด = คิดว่าเครื่องมือใหม่จะแก้ปัญหาทุกอย่าง**

ทักษะที่ใช้ได้ทั้งใน cloud + ในชีวิต:

1. **Configuration matters more than tool**
2. **Default settings rarely safe**
3. **Continuous monitoring** — set up เสร็จไม่จบ
4. **Periodic audit** — ของที่ไม่ใช้ → ลบ
5. **Least privilege everywhere** — ในระบบ + ในความสัมพันธ์
6. **Defense in depth** — หลายชั้น

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไปเราเข้าสู่ Part 6 — **AI & LLM Security** — ภัยใหม่ที่ dev ในปี 2026 ต้องเข้าใจ — เพราะ AI เปลี่ยนทั้งวิธีทำงานและวิธีโจมตี

---

## อภิธานศัพท์ (Glossary)

- **IaaS / PaaS / SaaS:** Infrastructure / Platform / Software as a Service
- **Shared Responsibility Model:** การแบ่งความรับผิดชอบระหว่าง provider กับ customer
- **IAM (Identity and Access Management):** ระบบจัดการ identity + permissions
- **IAM User:** identity สำหรับมนุษย์
- **IAM Role:** identity สำหรับ service / app — ไม่มี long-term credentials
- **IAM Policy:** เอกสาร JSON ระบุ permissions
- **Service Control Policy (SCP):** policy ที่ AWS Organizations applies guardrails
- **Permission Boundary:** maximum permissions ที่ user/role สามารถมี
- **MFA:** Multi-Factor Authentication (จากบทที่ 5)
- **Root Account:** บัญชีที่ create AWS account — ลูกกุญแจหลัก
- **Access Key:** long-term credential สำหรับ programmatic access
- **STS (Security Token Service):** ออก temp credentials
- **CloudTrail / Cloud Audit Logs / Activity Log:** บันทึก API calls
- **Security Group:** firewall rule สำหรับ EC2/VPC
- **NACL (Network ACL):** firewall ที่ subnet level
- **VPC (Virtual Private Cloud):** isolated network ใน AWS
- **GuardDuty:** AWS threat detection service
- **Security Hub:** AWS unified security findings
- **Config:** AWS service ติดตาม resource configuration
- **CSPM (Cloud Security Posture Management):** tool ที่ scan cloud config
- **CWPP (Cloud Workload Protection Platform):** runtime protection สำหรับ workload
- **Container:** ระบบ packaging app + dependencies
- **Distroless:** container image ที่ไม่มี OS shell / utilities
- **CIS Benchmark:** มาตรฐาน security configuration
- **IaC (Infrastructure as Code):** infrastructure ที่กำหนดด้วย code

---

## สรุป

1. **Shared Responsibility Model** — provider ดูแล "of cloud", คุณดูแล "in cloud"
2. **IAM = หัวใจของ cloud security** — least privilege, no root, no long-lived keys
3. **S3 public + open security groups** = แหล่งที่สุดของ breach
4. **IaC scanning** ก่อน deploy — Checkov, tfsec
5. **CloudTrail + GuardDuty + Security Hub** = visibility + detection
6. **Container security** — distroless, non-root, scanning, multi-stage builds
7. **Kubernetes** — pod security context, network policies, RBAC
8. **Audit IAM regularly** — Prowler, Steampipe, IAM Access Analyzer
9. **Encryption at rest + in transit** เป็น minimum
10. **Cloud ≠ secure by default** — configure ให้ถูก

ในบทถัดไป Part 6 เริ่มที่ **AI & LLM Security** — เมื่อ AI กลายเป็นทั้ง tool และ vulnerability ที่ dev ทุกคนต้องรู้

— Claude Opus 4.6
