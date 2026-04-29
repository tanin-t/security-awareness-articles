# บทความที่ 13: Dependency & Supply Chain Attack — เมื่อสิ่งที่คุณ Trust กลับทำร้ายคุณ

**CyberSecurity Awareness Series — Part 5: Developer-Specific**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของ Backdoor ที่เกือบเปลี่ยน Internet ทั้งโลก

ปลายเดือนมีนาคม 2024 — วิศวกร Microsoft ชื่อ **Andres Freund** กำลังทำงานวันธรรมดา ตรวจ benchmark ของ PostgreSQL บน Linux test system สังเกตเห็นว่า **SSH login** ใช้ CPU มากกว่าปกติ — แค่ **0.5 วินาที** ที่นานกว่า

คนส่วนใหญ่จะข้าม "noise" นี้ไป — แต่ Freund **ตามจี้** เรื่องนี้ ไล่ลงไปลึกๆ ใน library `liblzma` ที่เป็นส่วนของ `xz-utils` ที่ Linux distro ทุกตัวใช้

สิ่งที่เขาเจอคือ — **backdoor** ที่ฝังอยู่ใน xz-utils version 5.6.0 และ 5.6.1 ที่:

- เปิดให้ผู้โจมตี SSH login เป็น root **โดยไม่ต้องมี password / key ที่ถูกต้อง**
- ทำงานเฉพาะกับ key ที่ผู้โจมตี sign ด้วย private key เฉพาะ
- ฝัง code ผ่าน build process ที่ obfuscated มากๆ — ดูเหมือน test data
- ผ่านการ commit เป็นทีละนิดๆ ตลอด **2 ปี** ที่ผู้พัฒนาคนใหม่ (ชื่อ "Jia Tan") ค่อยๆ เพิ่มอำนาจในโครงการ

ถ้า backdoor นี้ไม่ถูกค้นพบก่อน — มันจะอยู่ใน Linux distro ส่วนใหญ่ (Debian, Ubuntu, Fedora, etc.) ที่กำลังจะ release ในอีกไม่กี่สัปดาห์ → **server ทั่วโลกหลายล้านเครื่อง** จะถูก backdoor

[อ่านรายละเอียดที่ Wikipedia](https://en.wikipedia.org/wiki/XZ_Utils_backdoor) / [Detailed analysis](https://research.swtch.com/xz-script)

ผู้โจมตี (เชื่อกันว่าเป็น state-sponsored, อาจเป็น Russia/China/NK) ใช้เวลา **3 ปี+** ในการ:

1. **สร้างตัวตนปลอม** — บัญชี GitHub ของ "Jia Tan"
2. **Contribute เล็กๆ น้อยๆ** ให้ open source projects เพื่อสร้างความเชื่อใจ
3. **Pressure ผู้พัฒนาเดิม** ของ xz-utils ให้ "หาคนช่วย"
4. **ใช้ sock puppet accounts** อีกหลายตัวเพื่อกดดันให้ Jia Tan ได้ maintainer status
5. **ใส่ backdoor** ที่ obfuscated มากๆ — แทบไม่มีทาง detect ด้วย code review ปกติ

นี่คือ **supply chain attack** ที่ซับซ้อนที่สุดที่เคยเห็นในประวัติศาสตร์ — และเกือบสำเร็จ

> **Open source ที่เราใช้ทุกวัน อาจมีคนแบบนี้อยู่ในนั้น — เราแค่ยังไม่เจอ**

---

## Supply Chain Attack คืออะไร

**Supply chain** = โซ่อุปทาน — ทุกขั้นตอนที่ "สิ่งหนึ่ง" เดินทางจากต้นทางมาถึงปลายทาง

ในวงการ software:

```
Developer code 
  ← Library / Package
    ← Dependency ของ Library
      ← Compiler / Build tool
        ← Container base image
          ← OS / Hardware
```

ทุกขั้นตอน เป็น **point of trust** — และเป็น **point of attack**

**Supply Chain Attack** = โจมตีผ่าน **trust chain** — ไม่ได้ตรงกับเป้าหมาย แต่ผ่านสิ่งที่เป้าหมายเชื่อถือ

### ทำไม Supply Chain ถึงเป็นเป้าโจมตีอันดับต้นๆ ในยุคนี้

1. **Modern software ใช้ dependency เยอะ** — Node.js project เฉลี่ย 1,500+ packages, Python 200+, Go 100+
2. **Trust by default** — เราติดตั้ง package โดยไม่อ่าน code
3. **Auto-update** — บางคนตั้ง auto-upgrade → malicious version ติดตั้งเอง
4. **One-to-many** — แฮก library ตัวเดียว = กระทบ project ที่ใช้ทั้งหมด
5. **ยากที่จะ detect** — code อยู่ลึกใน dependency ไม่มีใครอ่าน

### Anatomy ของ Supply Chain Attack

```
[Attacker] ─┐
            ├─→ [Trusted source] ─→ [Library / Tool]
            │                              ↓
            │                    [Project ของ developer]
            │                              ↓
            │                    [Production system]
            │                              ↓
            └────────────────→ [Attacker เข้าถึง production]
```

ผู้โจมตีไม่ต้อง break ทุกระบบ — แค่ break จุดเดียวที่ทุกคนเชื่อใจ

---

## กรณีศึกษาสำคัญ

### Case 1: SolarWinds (2020) — แม่ทุกของ supply chain attacks

**SolarWinds** เป็นบริษัทที่ทำ network monitoring software ชื่อ **Orion** ใช้กันใน:
- US Government (Treasury, State Department, DHS, ฯลฯ)
- Fortune 500 บริษัทหลายร้อย
- ทั่วโลก ~30,000 องค์กร

ในเดือนกันยายน 2020 — กลุ่ม APT29 (Cozy Bear, รัสเซีย) แฮกเข้าระบบของ SolarWinds **build server** และฝัง backdoor ลงใน Orion update — เรียกว่า **SUNBURST**

จากนั้น:

1. SolarWinds ส่ง update ตามปกติ — ไม่รู้ว่ามี backdoor
2. ลูกค้าทั้ง 30,000 บริษัท install update
3. ใน update มี SUNBURST malware
4. ผู้โจมตีเลือก ~100 องค์กรที่น่าสนใจ → ใช้ backdoor เข้าระบบ
5. Operation อยู่นาน **9 เดือน** ก่อนถูกค้นพบ (ธันวาคม 2020 โดย FireEye)

**ความเสียหาย:**

- US Government หลาย agency ถูก compromise
- Microsoft, Cisco, Intel, FireEye ถูก compromise
- Source code ของ Microsoft รั่ว
- Cleanup cost ประมาณ **$100 billion** ทั่วระบบเศรษฐกิจ

[อ่านรายละเอียด SolarWinds](https://en.wikipedia.org/wiki/SolarWinds#2019%E2%80%932020_supply_chain_attack)

**บทเรียน:** Software ที่คุณ install จาก trusted vendor — อาจมาพร้อมของแถม

### Case 2: event-stream (npm, 2018) — Library ที่ "เปลี่ยนมือ"

**event-stream** เป็น npm package ที่ใช้กันมาก (~2 ล้าน downloads/week)

ผู้พัฒนาเดิมไม่อยาก maintain แล้ว → ใครคนหนึ่งใน GitHub ขอเป็น maintainer — ผู้พัฒนาเดิม **ส่งสิทธิ์ให้** โดยไม่ verify ตัวตน

ผู้โจมตี (ภายใต้ชื่อ `right9ctrl`) ทำตามนี้:

1. รับ maintainer ของ `event-stream`
2. Add dependency ใหม่ชื่อ `flatmap-stream` — version แรก clean
3. ไม่กี่สัปดาห์ต่อมา — ปล่อย update ของ `flatmap-stream` ที่มี **malicious code**
4. Code ตรวจสอบ — ถ้า running ใน Bitcoin wallet ชื่อ Copay → ขโมย private keys

ใช้เวลา **2 เดือน** ก่อนถูกค้นพบ

[อ่าน post-mortem](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident.html)

**บทเรียน:** "Trusted maintainer" อาจเปลี่ยนคนได้ — เชื่อใจที่ project ไม่เพียงพอ

### Case 3: Codecov (2021) — เครื่องมือทดสอบที่ขโมย Secrets

**Codecov** เป็นบริการ code coverage ที่ developer ใช้กัน:

```bash
# ติดตั้งใน CI
bash <(curl -s https://codecov.io/bash)
```

ในเดือนเมษายน 2021 ผู้โจมตีพบช่องโหว่ใน Docker image creation ของ Codecov — ใช้แก้ไข `codecov-bash` script ใน CDN

ตั้งแต่ **มกราคม 2021** เป็นต้นมา — ทุกครั้งที่มี CI ใช้ codecov-bash:

1. Script เก็บ environment variables ของ CI (ที่มี API keys, tokens, credentials)
2. ส่งไปยัง server ของผู้โจมตี

ใช้เวลา **3 เดือน** ก่อนถูกค้นพบ — บริษัท ~29,000 บริษัทถูกกระทบ

[Codecov advisory](https://about.codecov.io/security-update/)

**บทเรียน:** Tool ที่คุณใช้ใน CI — มี access สูงเท่ากับ application ของคุณ

### Case 4: xz-utils (2024) — ที่กล่าวข้างต้น

ที่กล่าวต้นบทความ — **slow burn social engineering attack** ที่ใช้เวลา 3 ปี

**บทเรียน:**

- Attacker มี patience สูงมาก — รอได้หลายปี
- Open source maintainer พิเศษเพราะ "ไม่มีใครจ่าย"
- Burnt out maintainer = supply chain risk
- Build system attack ยากที่จะ detect ด้วย code review

### Case 5: PyPI / npm Typosquatting (ต่อเนื่อง)

**Typosquatting** = สร้าง package ปลอมชื่อคล้ายของจริง

ตัวอย่างที่เคยเกิด:

| ของจริง | ของปลอม |
|---|---|
| `requests` | `request`, `requets`, `reuqests` |
| `numpy` | `numyp`, `nupmy` |
| `python-dateutil` | `python-dateutil2` (different version) |
| `urllib3` | `urlib3`, `urllib` |
| `colorama` | `coloruma`, `colorrama` |

Dev ที่พิมพ์ผิด → install package ปลอม → malware เริ่มทำงาน

**ความถี่:** ตรวจพบ **300+ malicious packages/วัน** ใน PyPI/npm ในปี 2024 ([Sonatype State of the Software Supply Chain](https://www.sonatype.com))

### Case 6: ua-parser-js (2021) — Maintainer Account Hijack

**ua-parser-js** เป็น library ที่ใช้ใน Facebook, Apple, Microsoft, Amazon, Reddit ฯลฯ — 7 ล้าน downloads/week

ตุลาคม 2021 — maintainer ของ ua-parser-js โดน account hijack:

1. ผู้โจมตี (ที่ได้ npm credentials ของ maintainer) push 3 versions ใหม่
2. แต่ละ version มี malware ที่ steal credentials + cryptojacking
3. ใช้เวลา 4 ชั่วโมงก่อนถูกค้นพบ + revoke

**บทเรียน:** 2FA ของ maintainer = mandatory — หลังจากนี้ npm บังคับ 2FA สำหรับ popular packages

---

## Typosquatting — Package ปลอมชื่อคล้ายของจริง

### กลยุทธ์ที่ใช้กัน

#### 1. Letter Swap

```
requests → reqeusts
numpy → numyp  
django → djnago
```

#### 2. Letter Removal

```
requests → request
urllib3 → urlib3
```

#### 3. Letter Addition

```
requests → requeests
numpy → numpyy
```

#### 4. Hyphen / Underscore

```
python-dateutil → pythondateutil
sql_alchemy → sqlalchemy (จริง vs ปลอม)
```

#### 5. Different Registry

```
pip install requests   ← PyPI (real)
npm install requests   ← npm (อาจไม่ใช่ของจริง)
```

#### 6. Brand Confusion

```
react → react-dom (จริง)
react → reactjs (ปลอม)
```

### Defense

- **Copy-paste จาก official docs** เสมอ — ไม่พิมพ์เอง
- **ดู download count + age** — ของจริง downloads เยอะ + อายุนาน
- **ดู repo + author** — ของจริงมี repo ที่ active
- **Lock files** — ใส่ exact version + checksum

---

## วิธีป้องกัน — Practical Steps

### 1. Lock Files

ทุก package manager มี lock file:

| Language | Manifest | Lock |
|---|---|---|
| Node.js | `package.json` | `package-lock.json` หรือ `yarn.lock` หรือ `pnpm-lock.yaml` |
| Python | `requirements.txt` / `pyproject.toml` | `Pipfile.lock` / `poetry.lock` / `requirements.txt` (frozen) |
| Ruby | `Gemfile` | `Gemfile.lock` |
| Rust | `Cargo.toml` | `Cargo.lock` |
| Go | `go.mod` | `go.sum` |
| PHP | `composer.json` | `composer.lock` |

**Always commit lock files**:

- ✅ Reproducible builds (เครื่องคนอื่น install ได้ versions เดียวกันเป๊ะ)
- ✅ Pin transitive dependencies (deps ของ deps)
- ✅ Includes integrity checksums (npm), hashes (pip)

```json
// package-lock.json — มี integrity hash
{
  "dependencies": {
    "lodash": {
      "version": "4.17.21",
      "integrity": "sha512-..."
    }
  }
}
```

### 2. Pin Versions

```json
// ❌ Don't — accept any version
"react": "*"

// ⚠️ Range — auto-upgrades minor/patch
"react": "^18.0.0"  // 18.x.x
"react": "~18.1.0"  // 18.1.x

// ✅ Exact pin
"react": "18.2.0"
```

หรือใช้ tools ที่ pin auto:

- **Renovate** — auto PR สำหรับ updates (review ก่อน merge)
- **Dependabot** (GitHub) — same as Renovate

### 3. Vulnerability Scanning

#### npm

```bash
# Scan dependencies
npm audit

# Auto-fix
npm audit fix

# Production-only
npm audit --omit=dev
```

#### pip (Python)

```bash
# pip-audit
pip install pip-audit
pip-audit

# Or safety
pip install safety
safety check
```

#### Snyk (multi-language)

```bash
npm install -g snyk
snyk auth
snyk test       # scan project
snyk monitor    # ติดตาม + alert
```

#### GitHub Dependabot

ใน Settings → Code security → Dependabot:
- ✅ Dependency graph
- ✅ Dependabot alerts
- ✅ Dependabot security updates
- ✅ Dependabot version updates

→ GitHub แจ้งเตือนเมื่อมี vulnerability ใน deps + เปิด PR auto

#### Other

- **Trivy** — scan container images + dependencies
- **Grype** — vulnerability scanner
- **OWASP Dependency-Check**
- **Sonatype Nexus IQ**

### 4. Review Before Update

เมื่อ Dependabot / Renovate เปิด PR — **อย่ากด merge เลย**:

1. **อ่าน changelog** — มี breaking change ไหม?
2. **ดู release date** — published เมื่อไหร่? ของใหม่มาก = เสี่ยง
3. **ดู contributor** — มี contributor ใหม่ไหม?
4. **ดู published-by** — maintainer คนเดิมไหม?
5. **Check vulnerability database** — มี CVE สำหรับ version ใหม่ไหม?
6. **Test ก่อน merge** — CI ผ่านหรือเปล่า?

### 5. Verify Checksums

```bash
# Download package
wget https://example.com/package.tar.gz

# Compare checksum กับที่ vendor publish
echo "expected_sha256_hash package.tar.gz" | sha256sum --check
```

ใน package manager ส่วนใหญ่ — ทำให้อัตโนมัติด้วย lock file

### 6. Use Trusted Registry

#### Internal Mirror

แทนที่จะ pull จาก npm/PyPI ตรง — ใช้ internal mirror ที่ verify แล้ว:

- **Sonatype Nexus** — internal artifact repository
- **JFrog Artifactory**
- **AWS CodeArtifact**
- **GitHub Packages**

```bash
# .npmrc
registry=https://nexus.company.com/repository/npm/
```

ข้อดี:
- Cache packages ในบริษัท
- Block packages ที่ไม่อนุญาต
- Scan ก่อนปล่อยให้ download
- Audit log การใช้งาน

### 7. Network Egress Control

ใน production environment — **block outbound network** ที่ไม่จำเป็น:

```yaml
# Kubernetes NetworkPolicy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
spec:
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/8       # internal
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 169.254.169.254/32 # block metadata service
```

→ แม้ malicious dependency ถูก install — ก็ส่งข้อมูลออกไม่ได้

---

## Software Bill of Materials (SBOM)

### SBOM คืออะไร

**SBOM** = "list ของส่วนประกอบทั้งหมดของ software" — เหมือน "bill of materials" ของ manufacturing

```yaml
# ตัวอย่าง SBOM (CycloneDX format)
name: my-app
version: 1.0.0
components:
  - type: library
    name: lodash
    version: 4.17.21
    purl: pkg:npm/lodash@4.17.21
    licenses:
      - id: MIT
    hashes:
      - alg: SHA-256
        content: abc123...
  - type: library
    name: express
    version: 4.18.2
    ...
```

### ทำไมต้องมี SBOM

#### 1. Vulnerability Response

ตอนที่มี CVE ใหม่ — บริษัทต้องตอบได้ใน 5 นาทีว่า "เราใช้ component ที่กระทบไหม?"

ตัวอย่าง — ตอน **Log4Shell** ปะทุในธันวาคม 2021:
- บริษัทที่มี SBOM → ตอบได้ในชั่วโมง
- บริษัทที่ไม่มี → ใช้เวลาหลายวัน-สัปดาห์ในการ inventory

#### 2. License Compliance

- รู้ว่าใช้ license อะไรบ้าง (MIT, Apache, GPL, ฯลฯ)
- กัน license violation (เช่น GPL ใน proprietary code)

#### 3. Regulatory

- US Executive Order 14028 (2021) require SBOM สำหรับ federal vendors
- EU Cyber Resilience Act 2024 — SBOM required
- มาตรฐาน NIST, ISO 27001 ก็เริ่มอ้างอิง SBOM

### Generate SBOM

#### Syft (multi-language)

```bash
# ติดตั้ง
brew install syft

# Generate SBOM
syft my-app/ -o cyclonedx-json > sbom.json
```

#### Format มาตรฐาน

- **CycloneDX** (OWASP) — popular ใน security
- **SPDX** (Linux Foundation) — popular ใน licensing

#### Integration

```yaml
# GitHub Actions
- name: Generate SBOM
  uses: anchore/sbom-action@v0
  with:
    artifact-name: sbom.spdx.json
```

### Use SBOM

#### Vulnerability Scanning ที่ละเอียด

```bash
# Grype scan SBOM
grype sbom:sbom.json
```

#### Continuous Monitoring

```bash
# Anchore Grype + alert
grype sbom:sbom.json --output json | \
  jq '.matches | select(.vulnerability.severity == "Critical")' | \
  alert-to-slack.sh
```

---

## Hands-on: Audit Project ของคุณ

### Step 1: Inventory Dependencies

```bash
# Node.js
npm list --all > deps.txt
npm list --depth=0  # top-level only

# Python
pip list > deps.txt
pip-tree

# Go
go list -m all
```

ดูจำนวน — มักจะตกใจที่เยอะกว่าที่คิด

### Step 2: Run Vulnerability Scan

```bash
# npm
npm audit --omit=dev

# pip
pip-audit

# Snyk
snyk test --severity-threshold=high
```

### Step 3: Review Top Dependencies

ดู top 20 packages ที่ใหญ่/มี dependents เยอะ:

```bash
# npm
npm ls --depth=0 --long

# Look for:
# - Maintainer คนล่าสุด
# - Last published date
# - Open issues
# - License
```

ถ้า package ที่ใหญ่:
- **Inactive maintenance** (ไม่มี commit ใน 1 ปี) → เสี่ยง (อาจถูก hijack)
- **Single maintainer** → เสี่ยง
- **Hundreds of unresolved issues** → เสี่ยง

### Step 4: Check Lock Files

```bash
# Confirm lock file committed
git log -- package-lock.json

# Verify integrity
npm ci  # ใช้ exact version จาก lock — fail ถ้าไม่ match
```

### Step 5: Setup Automated Scanning

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
    
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Step 6: Setup CI Check

```yaml
# .github/workflows/security.yml
name: Security Scan
on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run npm audit
        run: npm audit --audit-level=high
      
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      
      - name: Generate SBOM
        uses: anchore/sbom-action@v0
```

### Step 7: Restrict Outbound Egress

ใน production:

```yaml
# Network policy ที่จำกัด egress
# Block direct internet access
# Allow only specific internal services + API endpoints
```

---

## Action Items

### วันนี้

- [ ] **รัน `npm audit` / `pip-audit`** บน project ทุกตัว
- [ ] **Confirm lock files** ถูก commit
- [ ] **Enable Dependabot** บน GitHub repos

### สัปดาห์นี้

- [ ] **List top dependencies** + check maintainer status
- [ ] **Setup Snyk หรือ Trivy** ใน CI
- [ ] **Generate SBOM** สำหรับ project หลัก
- [ ] **Pin critical dependencies** เป็น exact version

### เดือนนี้

- [ ] **Setup internal artifact mirror** (Nexus / Artifactory)
- [ ] **Network egress policy** สำหรับ production
- [ ] **Automated PR review process** สำหรับ Dependabot updates
- [ ] **Onboarding checklist** เรื่อง dependencies

### สำหรับ Tech Lead / DevSecOps

- [ ] **SBOM generation** ใน build pipeline
- [ ] **Vulnerability monitoring** + alerting
- [ ] **Dependency policy document** — อะไรอนุญาต/ห้าม
- [ ] **Vendor security assessment** — สำหรับ critical dependencies
- [ ] **Incident response plan** สำหรับ supply chain attack
- [ ] **Quarterly dependency review** — audit + clean up

### สำหรับ CTO / Engineering Manager

- [ ] **Procurement policy** — vendor security review ก่อนซื้อ
- [ ] **Open source contribution policy** — ทีมต้อง verify identity
- [ ] **Budget for security tools** — Snyk, Sonatype, etc.
- [ ] **Insurance review** — supply chain attack ครอบคลุมไหม

---

## บทเรียนชีวิตจากบทความนี้

> **Trust แต่ Verify — เชื่อใจได้ แต่ต้องตรวจสอบเสมอ**

วลี "Trust but verify" มาจาก **Ronald Reagan** ปี 1987 ตอนเจรจา nuclear treaty กับ USSR — เขาเรียนภาษิต Russian "doveryai, no proveryai" แล้วเอามาใช้ พอ Gorbachev ได้ยินก็หัวเราะ "You repeat that at every meeting" — Reagan ตอบ "I like it"

ในชีวิตจริง — หลักการนี้ใช้กับทุกเรื่อง:

#### ในการเงิน

- เซ็น contract → **อ่านทุก page**
- ลงทุนกับ "เพื่อนสนิท" → ดู track record + เอกสาร
- เปิดบัญชีกับธนาคาร "ยอดเยี่ยม" → ตรวจ rate ทุกธนาคาร

#### ในการทำงาน

- รับ candidate ที่ "best on paper" → reference check, technical interview
- ทำธุรกิจกับ supplier "เชื่อใจได้" → audit รายปี
- เซ็นสัญญากับ vendor "เก่าแก่" → ดู financials + reviews

#### ในความสัมพันธ์

- คนที่ดูดีในที่ทำงาน — อาจไม่เหมือนใน private life
- เพื่อนที่สนิท — อาจมีมุมที่คุณไม่เห็น
- คนรัก — actions ตรงกับ words ไหม?

#### ในข้อมูล

- ข่าวที่ "เพื่อน share" → check source
- โฆษณา "Doctor recommended" → ใครคือ doctor นั้น?
- "ข้อมูล vital" จาก Line group → fact check

> **Trust แบบไม่มีเงื่อนไข = ตกหลุมพราง**  
> **Trust แบบ verify = ปลอดภัยที่ใช้ได้จริง**

ในเรื่อง software:

- **Library popular** = เชื่อได้... แต่ verify maintainer + version
- **Cloud provider ใหญ่** = เชื่อได้... แต่ verify configuration
- **Vendor มี SOC 2** = ดี... แต่ verify scope ของ audit
- **Code review เพื่อนร่วมทีม** = ดี... แต่ verify automated tests

> **คนที่ verify เป็นนิสัย — ผ่าน supply chain attack ได้ง่ายกว่า**

นี่ไม่ใช่ paranoia — แต่เป็น **due diligence**

> **คนเก่งจริง รักการตรวจสอบ — เพราะพวกเขารู้ว่าทุกอย่างผิดพลาดได้**

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไป เราจะพูดถึง **Secure Coding Practices** — เขียน code แบบที่รู้ว่ามีคนพยายาม hack — รวมถึง OWASP Top 10, input validation, common vulnerabilities ที่ dev ทำพลาดบ่อย

---

## อภิธานศัพท์ (Glossary)

- **Supply Chain Attack:** โจมตีผ่าน trusted third-party
- **Dependency:** library / package ที่ project ใช้
- **Transitive Dependency:** dependency ของ dependency
- **Lock File:** ไฟล์ที่ระบุ exact version + checksum ของ deps
- **Pinning:** ระบุ version แน่นอน (vs range)
- **Typosquatting:** package ปลอมชื่อคล้ายของจริง
- **Package Hijacking:** การยึด maintainer account ของ package
- **SBOM (Software Bill of Materials):** list ของ component ทั้งหมดใน software
- **CVE (Common Vulnerabilities and Exposures):** ฐานข้อมูล vulnerability
- **CVSS (Common Vulnerability Scoring System):** ระบบให้คะแนน severity ของ vulnerability
- **Dependabot:** GitHub tool ที่ alert + เปิด PR สำหรับ updates
- **Snyk / Trivy / Grype:** vulnerability scanners
- **Renovate:** alternative ของ Dependabot — flexible กว่า
- **CycloneDX / SPDX:** SBOM formats
- **Artifact Repository:** internal mirror ของ packages (Nexus, Artifactory)
- **Egress Control:** จำกัด outbound traffic
- **APT (Advanced Persistent Threat):** กลุ่ม attacker ที่ skilled + persistent
- **Watering Hole:** แฮกเว็บที่เป้าหมายชอบเข้า
- **Sock Puppet:** account ปลอมที่ใช้สนับสนุนตัวเอง

---

## สรุป

1. **Supply chain attack ผ่าน "trust"** — library, vendor, build system ที่คุณเชื่อใจ
2. **Cases สำคัญ:** SolarWinds (2020), event-stream (2018), Codecov (2021), xz-utils (2024)
3. **Typosquatting** เป็น attack ที่ยังเกิดทุกวัน — copy-paste จาก official docs เสมอ
4. **Lock files + pin versions + integrity checks** = พื้นฐานที่ต้องทำ
5. **Vulnerability scanning** อัตโนมัติ — Dependabot, Snyk, npm audit
6. **Review ก่อน update** — อย่า blindly upgrade
7. **SBOM** เป็นมาตรฐานใหม่ — ต้องรู้ว่าคุณใช้อะไรบ้าง
8. **Network egress control** = ป้องกัน data exfiltration ถ้า dependency หลุด
9. **Internal artifact mirror** สำหรับ enterprise — control + audit
10. **Trust + Verify** = ทักษะที่ใช้ได้ทั้งใน software และในชีวิต

ในบทถัดไป — **Secure Coding Practices** — เขียน code อย่างไรให้ปลอดภัย รวมถึง OWASP Top 10, common vulnerabilities, defensive coding patterns

— Claude Opus 4.6
