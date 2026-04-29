# บทความที่ 19: Zero Trust — อย่าเชื่อใจใคร แม้แต่คนในเครือข่ายเดียวกัน

**CyberSecurity Awareness Series — Part 8: Advanced Concepts**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของ Google ที่ "ทิ้ง VPN ทั้งบริษัท"

ปลายปี 2009 — **Google** ถูก attack ที่เรียกว่า **"Operation Aurora"** กลุ่มแฮกเกอร์จากจีน (อ้างเชื่อมโยงกับรัฐบาล) เจาะระบบ Google + อีก 30+ บริษัท (Adobe, Yahoo, Symantec, Northrop Grumman) ผ่าน:

1. **Phishing email** ที่มี link ไปยังเว็บ malicious
2. Exploit zero-day ใน Internet Explorer
3. Install malware บนเครื่อง employee
4. **เข้า internal network** ของ Google ผ่าน VPN ของ employee นั้น
5. Move laterally ภายใน → ขโมย source code + access สำคัญ

ที่น่าตกใจที่สุด — เมื่อแฮกเกอร์เข้า internal network ของ Google ได้แล้ว — **ทุกอย่างเปิดให้** เพราะ Google (เหมือนบริษัทอื่นในยุคนั้น) ใช้ **castle-and-moat security**:

- ข้างนอก network = ห้าม
- ข้างใน network = trust ทุกอย่าง

[อ่านรายละเอียด Operation Aurora](https://en.wikipedia.org/wiki/Operation_Aurora)

หลังเหตุการณ์ — Google **เปลี่ยนแนวคิดทั้งบริษัท** มาใช้ **Zero Trust** ในชื่อโครงการ **"BeyondCorp"** (2014) — ทิ้ง VPN ออกหมด แทนที่ด้วย:

- ทุก request ต้อง authenticate + authorize
- ไม่มี "internal network" ที่ trusted
- Device ต้อง verify ทุกครั้ง
- User ต้อง verify ทุกครั้ง
- Resource access แตกตามต้องการ

ผลคือ — Google security เพิ่มขึ้นมหาศาล + employees ทำงานจากที่ไหนก็ได้โดยไม่ต้อง VPN ([BeyondCorp paper](https://research.google/pubs/pub43231/))

นี่คือจุดเริ่มต้นของ **Zero Trust Architecture** — model ที่บริษัทใหญ่ทั่วโลกค่อยๆ adopt ในทศวรรษถัดมา

> **"Never trust, always verify"**

ในบทความนี้เราจะดู:

1. **ทำไม castle-and-moat ใช้ไม่ได้แล้ว**
2. **Zero Trust คืออะไร** + 3 หลักการ
3. **BeyondCorp** เป็น case study
4. **Practical implementation** สำหรับ dev team
5. **เริ่มต้น** ที่ไหน

---

## Traditional Security: Castle-and-Moat

### Model เก่า — "Walls and Gates"

ในยุค 1990s-2000s — บริษัทคิดเรื่อง security เหมือน **ปราสาท**:

```
[Internet (อันตราย)]
       ↓
   [Firewall / DMZ]   ← gate
       ↓
   [Internal Network] ← ภายในกำแพง
       ↓
   [Servers, databases, file shares] ← ทุกอย่าง trusted
```

หลักการ:

- **ภายนอก** (internet) = อันตราย → block ที่ firewall
- **ภายใน** (corporate network) = ปลอดภัย → trust by default
- **VPN** = ทางเข้ากำแพง สำหรับ employees ที่ทำงาน remote

### ทำไม Model นี้ไม่ Work อีกแล้ว

#### 1. Perimeter หายไป

```
ในอดีต:
 - 100% ของ workload อยู่ใน data center บริษัท
 - 100% ของ employees ทำงานในออฟฟิศ
 - Devices ทั้งหมดเป็นของบริษัท

ปัจจุบัน (2026):
 - 80%+ ของ workload อยู่บน cloud (AWS, GCP, Azure)
 - 50%+ ของ employees ทำ hybrid / remote
 - BYOD - personal devices ใช้กับงาน
 - SaaS apps (Slack, Notion, Salesforce) อยู่ที่ไหนก็ไม่รู้
```

→ **"Castle" ไม่มีแล้ว** — workload + users กระจายไปทุกที่

#### 2. Insider Threat

ถ้าใครเข้า "ภายใน" ได้ — **ทุกอย่างเปิด**:

- Phishing → เครื่อง employee 1 ติด → เข้าทุกระบบได้
- พนักงานทุจริต → เข้าถึงข้อมูลที่ไม่ควร
- พนักงานเก่า → access ยังไม่ถูก revoke

**60% ของ data breach** เริ่มจาก credential ของ insider (จาก Verizon DBIR 2024)

#### 3. Lateral Movement

แฮกเกอร์ที่เข้าได้ จะเดินภายใน network หา target ที่ต้องการ:

```
[Phishing] → [Employee laptop] → 
[Domain controller] → [File server] → 
[Database server] → [Crown jewels]
```

ใน castle-and-moat — กำแพงเดียว ผ่านครั้งเดียว เข้าได้ทุกที่

#### 4. Cloud + SaaS

ถ้า company ใช้:
- AWS / GCP / Azure (workload)
- Microsoft 365 / Google Workspace (productivity)
- Slack (chat)
- Salesforce (CRM)
- Datadog (monitoring)

→ ไม่มี "ภายใน" / "ภายนอก" ที่ชัดเจนอีกต่อไป

#### 5. Modern Threats Bypass Perimeter

- Phishing — bypass firewall (legitimate website)
- Compromised vendor — supply chain (บทที่ 13)
- Stolen credentials — login เหมือน user จริง
- Social engineering — bypass technical controls

---

## Zero Trust — "Never Trust, Always Verify"

### หัวใจของ Zero Trust

> **ห้ามมี "trusted zone" ใน network ใดๆ — ทุก access request ต้อง verify ทุกครั้ง**

ไม่ว่า request มาจาก:
- Office Wi-Fi
- Home network
- Coffee shop
- VPN
- Cloud
- ภายใน server เดียวกัน

→ **Treat ทุกอย่างเป็น untrusted by default**

### 3 Foundational Principles

#### 1. Verify Explicitly

ทุก authentication / authorization decision ต้องอิงข้อมูล **ทุกอย่างที่มี**:

- **User identity** — who is this?
- **Device identity** — what device?
- **Device health** — patched? jailbroken? infected?
- **Location** — where from?
- **Time** — when?
- **Behavior** — typical pattern?
- **Resource** — what are they accessing?
- **Sensitivity** — how important?

→ ตัดสินใจ **per-request** ไม่ใช่ "trust ทั้ง session"

#### 2. Least Privilege Access

User / service ได้สิทธิ์ **เท่าที่จำเป็น เท่านั้น**:

- **Just-in-time** — ขอ access ตอนต้องใช้, expire after use
- **Just-enough** — minimum needed permissions
- **Adaptive** — เพิ่ม / ลด ตาม risk

#### 3. Assume Breach

ออกแบบระบบ **on the assumption** ว่า attacker อยู่ใน network แล้ว:

- **Microsegmentation** — ทุก zone แยกกัน
- **Encryption everywhere** — even internal traffic
- **Audit everything** — full visibility
- **Limit blast radius** — compromise of one ≠ compromise of all

---

## Zero Trust vs Traditional — Side by Side

| Aspect | Traditional (Castle-Moat) | Zero Trust |
|---|---|---|
| **Perimeter** | Network boundary | None — identity is the perimeter |
| **Inside network** | Trusted | Untrusted |
| **Authentication** | Once at login | Every request |
| **Authorization** | Coarse (role / group) | Fine-grained, per-request |
| **VPN** | Required for remote | Not needed (each app has its own auth) |
| **Lateral movement** | Possible | Limited (microsegmentation) |
| **Device trust** | "Domain joined" = OK | Continuous device posture check |
| **Encryption** | At perimeter | Everywhere (even internal) |
| **Visibility** | Limited (perimeter logs) | Full (every request logged) |
| **Default** | Allow internal | Deny by default |

---

## BeyondCorp — Google's Zero Trust Implementation

Google's approach (2011-onwards) ที่กลายเป็น blueprint สำหรับ industry:

### Key Components

#### 1. Device Inventory

ทุก device ที่ใช้กับ Google services:
- บันทึกใน inventory
- มี device certificate (X.509)
- Track posture (OS version, patches, encryption)

#### 2. User Inventory

User database ที่เป็น single source of truth — integrated กับ:
- HR system
- Group memberships
- Role assignments

#### 3. Trust Tiers

แบ่ง devices + users เป็น tiers ตาม trust level:

```
Tier 1 (highest): Production-managed device + corporate user
Tier 2: Managed device + corporate user
Tier 3: Personal device + corporate user
Tier 4: Untrusted
```

ตาม tier → ได้ access ระดับ resource ต่างๆ

#### 4. Access Proxy

**Identity-Aware Proxy (IAP)** — gate ที่ทุก request ผ่าน:

```
[User device] → [IAP] → [Backend service]
                  ↑
          ตัดสินใจที่นี่:
          - User authenticated?
          - Device trusted?
          - Tier sufficient?
          - MFA recent?
          - Location OK?
          - Behavior normal?
```

#### 5. Continuous Verification

ไม่ใช่ "login ตอนเช้า ใช้ทั้งวัน" — แต่ **re-verify ทุก request**

### Result

- **VPN ถูกทิ้ง** สำหรับ corporate apps
- **Remote work** เหมือนทำงานใน office (no setup)
- **Security เพิ่ม** มหาศาล
- **User experience ดีขึ้น** — ไม่ต้อง connect VPN แต่ละครั้ง

---

## Zero Trust Architecture Components

### 1. Identity Provider (IdP)

Single source of truth สำหรับ user identity:

- **Okta**, **Microsoft Entra ID** (formerly Azure AD), **Google Workspace**
- **Auth0**, **Keycloak**
- **Ping Identity**

→ Integrate ทุก app ผ่าน SSO (Single Sign-On)

### 2. Device Management (MDM)

ที่กล่าวในบทที่ 8 — device trust ต้อง:

- **Enrolled** ใน MDM
- **Compliant** กับ policy (encryption, OS version, etc.)
- **Healthy** (no malware, recent check-in)

### 3. Identity-Aware Proxy / Access Gateway

Gateway ที่ enforce policy:

- **Cloudflare Access** — popular ZTNA solution
- **Zscaler ZTNA**
- **Palo Alto Prisma Access**
- **AWS Verified Access**
- **Google Cloud IAP**
- **Tailscale** (for SSH, smaller scale)

### 4. Network Microsegmentation

Internal network แยกเป็น micro-segments:

- **VPC isolation** (cloud)
- **Network policies** (Kubernetes)
- **Service mesh** (Istio, Linkerd) สำหรับ service-to-service

### 5. Service-to-Service Authentication

ภายใน internal — services ต้อง auth กัน:

- **mTLS** (mutual TLS)
- **SPIFFE / SPIRE** — workload identity
- **Service mesh** features

### 6. Continuous Monitoring

- SIEM
- Behavioral analytics (UEBA)
- Anomaly detection
- Real-time risk scoring

### 7. Policy Engine

Centralized policy management:

```yaml
policy:
  resource: "production-database"
  required:
    - user.role == "dba"
    - device.compliant == true
    - device.tier >= 2
    - mfa.last_auth < 4h
    - request.location in ["TH", "SG"]
    - session.risk_score < 50
```

---

## Zero Trust สำหรับ Dev Team

### SSH Access — แทน VPN

#### Traditional

```
Dev → VPN → Bastion host → SSH → Production server
       ↑      ↑
   long-lived,  open port
   shared
```

#### Zero Trust

```
Dev → Identity proxy → SSH (with short-lived cert)
       ↑
   Verify:
   - User identity
   - Device trust
   - MFA
   - Approved access (ticket / on-call)
```

ใช้:
- **Teleport** — open source SSH proxy with Zero Trust
- **HashiCorp Boundary** — alternative
- **Tailscale SSH** — for smaller setups
- **AWS Systems Manager Session Manager** — no SSH key required

### Database Access

#### Traditional

```
Dev laptop → VPN → DB (with username + password เก็บใน manager)
```

→ Password อาจรั่ว, no audit, no per-query authorization

#### Zero Trust

```
Dev laptop → Identity proxy → 
   [Verify identity, device, MFA, ticket] → 
   Issue short-lived DB credentials → 
   DB
```

ใช้:
- **HashiCorp Vault** — dynamic database credentials
- **AWS RDS IAM authentication**
- **Boundary** — broker DB access
- **Teleport database access**

```bash
# Vault dynamic credential
vault read database/creds/readonly-role
# → username: v-token-readonly-xyz
#   password: ...
#   lease_duration: 1h
```

### API Access

ทุก API ภายใน — auth ทุก request:

```
Service A → Service B
   |
   ↓ JWT / mTLS / Access token
   ↓
   B verifies:
   - Identity of A (workload identity)
   - A allowed to call this endpoint
   - Token not expired
```

### CI/CD Pipeline

```
GitHub Actions / GitLab CI →
   OIDC token (short-lived) →
   AWS IAM role (assumes via OIDC) →
   Deploy
```

ไม่มี long-lived AWS access keys ใน CI

### Container Workload

```yaml
# k8s pod with workload identity
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  annotations:
    iam.gke.io/gcp-service-account: app@project.iam.gserviceaccount.com

# Pod uses SA → gets short-lived GCP token → access GCP resources
```

### Internal Tooling

- **Replace VPN-protected admin panels** → IAP-protected
- **Internal docs (Confluence/Notion)** → SSO + RBAC
- **Logs / monitoring** → identity-aware

---

## Implementation Roadmap

Zero Trust ไม่ใช่ "1 day project" — เป็น journey ที่ทำเป็นปี

### Phase 1: Foundation (Month 1-3)

- [ ] **Identity Provider** central (Okta / Entra)
- [ ] **MFA enforcement** ทุก user
- [ ] **SSO** สำหรับ apps สำคัญ
- [ ] **Device inventory** + MDM
- [ ] **User directory cleanup** — disable inactive accounts

### Phase 2: Network (Month 3-9)

- [ ] **VPC segmentation** ใน cloud
- [ ] **Network policies** ใน k8s
- [ ] **Service mesh** สำหรับ internal traffic
- [ ] **Microsegmentation** ของ critical assets

### Phase 3: Access (Month 6-12)

- [ ] **ZTNA solution** (Cloudflare Access / Zscaler)
- [ ] **SSH replacement** (Teleport / Boundary)
- [ ] **Database broker** (Vault dynamic credentials)
- [ ] **API authorization** with JWT / mTLS

### Phase 4: Continuous (Ongoing)

- [ ] **Behavioral analytics** (UEBA)
- [ ] **Risk-based policies**
- [ ] **Continuous compliance**
- [ ] **Regular audits**

### Phase 5: VPN Sunset (Final)

- [ ] **Remove VPN** สำหรับ user access (เมื่อ ZTNA เต็มที่)

---

## Common Pitfalls

### 1. ทำเร็วเกินไป

→ User experience แย่ → ทีมไม่ adopt → fall back to old way

### 2. ไม่ Train Team

→ Dev frustrated → workaround → security ไม่จริง

### 3. Tool-First, Not Strategy-First

→ ซื้อ tool แต่ไม่มี policy → tool ไม่ใช้ประโยชน์

### 4. Ignore Legacy Systems

→ Old apps ที่ไม่รองรับ ZT → loophole

### 5. Set and Forget

→ Policy ที่ไม่ update → drift → ไม่ work อีกแล้ว

---

## Action Items

### วันนี้

- [ ] **เช็ค VPN usage** — ใครยังใช้บ้าง? เพื่ออะไร?
- [ ] **MFA enforcement** — บังคับบน critical apps
- [ ] **List internal apps** ที่ accessible from internet (SSO ผ่านหรือยัง?)

### สัปดาห์นี้

- [ ] **Centralize identity** ผ่าน Okta / Entra
- [ ] **SSO ทุก app** ที่รองรับ
- [ ] **Audit IAM permissions** — remove excessive access
- [ ] **Setup MDM** สำหรับ company devices

### เดือนนี้

- [ ] **Pilot ZTNA solution** สำหรับ 1 app
- [ ] **Replace SSH bastion** ด้วย Teleport / similar
- [ ] **Database access broker** (Vault)
- [ ] **OIDC for CI/CD** — remove long-lived keys

### สำหรับ CISO / Security Lead

- [ ] **Zero Trust strategy document**
- [ ] **3-year roadmap**
- [ ] **Budget approval** for ZT tooling
- [ ] **Executive buy-in** — need C-level support
- [ ] **Change management plan** — communicate ทีม

### สำหรับ DevOps / Platform Team

- [ ] **Infrastructure for workload identity** (SPIFFE/SPIRE)
- [ ] **Service mesh** evaluation
- [ ] **mTLS** สำหรับ internal traffic
- [ ] **Microsegmentation** policy

---

## บทเรียนชีวิตจากบทความนี้

> **"อยู่ในกลุ่มเดียวกัน" ไม่ได้แปลว่าปลอดภัย — ตรวจสอบทุกครั้ง ทุกคน ทุกสถานการณ์**

ในชีวิตจริง — เรามักคิดว่า "อยู่กลุ่มเดียวกัน" = trustworthy:

#### ในการเงิน

- **เพื่อนร่วมงาน** ขอยืมเงิน — "เพื่อนกัน เชื่อได้"
- **ญาติ** ชวนลงทุน — "ครอบครัว ไม่หลอกแน่"
- **คนมีฐานะ** ขอให้ช่วยเซ็น — "เขาไม่จำเป็นต้องโกง"

→ ในความเป็นจริง — Pyramid scheme, family business fraud มีเยอะที่สุดในบรรดาประเภทอาชญากรรมทางการเงิน

#### ในที่ทำงาน

- **HR** ส่ง email ขอข้อมูล personal — "เป็น HR ของบริษัท ปลอดภัย"
- **CEO** ส่ง message ขอโอนเงิน — "ผู้บริหารใหญ่ คงเรื่องจริง"
- **IT helpdesk** โทรขอ password — "ทีม IT ของเรา ช่วยได้"

→ MGM Resorts hack (2023) ผ่าน social engineering เข้า IT helpdesk

#### ในความสัมพันธ์

- **คู่รัก** ขอรหัสผ่านบัญชี — "เขาไม่หลอกแน่นอน"
- **เพื่อนสนิท** ขอ access laptop — "เชื่อใจได้"
- **ครอบครัว** ใช้บัญชีร่วม — "ในครอบครัวกัน"

→ Romance scam, financial abuse ในความสัมพันธ์ — เพราะ trust ที่ "เกินจำเป็น"

#### ในข้อมูล

- **ข่าวจากคนรู้จัก** — share ต่อโดยไม่ check
- **คำแนะนำจาก senior** — ทำตามโดยไม่ verify
- **ข้อมูลจาก group LINE** — เชื่อเลย

→ Misinformation ที่ระบาด — เพราะ trust ที่ในกลุ่มเดียวกัน

> **Trust based on group ≠ Trust based on verification**

ในเรื่อง Zero Trust:

- **ในเดียวกับเรา** = เคยเป็นพอ
- **ปัจจุบัน** = ตรวจสอบทุกครั้ง

หลักการนี้ใช้ในชีวิตได้:

#### Principle 1: Verify Explicitly

ก่อนเชื่ออะไร — ถามตัวเอง:
- จาก source ไหน?
- มี evidence ไหม?
- ตรงกับข้อเท็จจริงอื่นไหม?
- มีอะไรน่าสงสัยไหม?

#### Principle 2: Least Privilege ใน Trust

ให้ trust **เท่าที่จำเป็น**:

- เพื่อนใหม่ → trust กับเรื่องเล็ก ก่อน
- พนักงานใหม่ → access น้อย ก่อน
- Partner ใหม่ → ไม่ทุก account ทันที

#### Principle 3: Assume Breach

คิดเผื่อว่า — **ทุกอย่างพังได้**:

- เพื่อนสนิทอาจเปลี่ยน
- พนักงานดี อาจถูกแฮก
- Partner ที่ดี อาจมี bad day

→ มี backup plan, redundancy, monitoring

> **"Trust แต่ Verify" ไม่ใช่ paranoia — เป็น maturity**

> **คนที่เชื่อใจคนทุกคน = naive**  
> **คนที่ไม่เชื่อใครเลย = lonely**  
> **คนที่ verify ก่อนเชื่อ = balanced**

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไป — **Threat Modeling** — เทคนิคที่ใช้คิดเป็น attacker เพื่อป้องกันเป็น defender — ใช้ได้ทั้งใน security และในการตัดสินใจชีวิต

---

## อภิธานศัพท์ (Glossary)

- **Zero Trust:** model ที่ "never trust, always verify"
- **Castle-and-Moat:** model เก่าที่ trust internal, deny external
- **Perimeter Security:** ป้องกันที่ขอบ network
- **Microsegmentation:** แบ่ง network เป็น zones เล็กๆ
- **ZTNA (Zero Trust Network Access):** alternative ของ VPN
- **Identity Provider (IdP):** central identity service
- **SSO (Single Sign-On):** login ครั้งเดียว ใช้ได้หลาย apps
- **MFA / 2FA:** multi-factor authentication
- **Identity-Aware Proxy (IAP):** gateway ที่ check identity ทุก request
- **Workload Identity:** identity สำหรับ services / pods
- **mTLS (mutual TLS):** ทั้งสองฝ่ายต้องมี certificate
- **SPIFFE / SPIRE:** standard + impl สำหรับ workload identity
- **Service Mesh:** infrastructure layer สำหรับ service-to-service communication
- **JIT Access:** Just-in-Time access — ขอตอนใช้, expire
- **JEA Access:** Just-Enough Access — minimum permissions
- **UEBA (User and Entity Behavior Analytics):** ML-based anomaly detection
- **BeyondCorp:** Google's Zero Trust implementation
- **Lateral Movement:** การเดินภายใน network หลังเข้าได้
- **Blast Radius:** ขอบเขตของความเสียหายเมื่อ compromise

---

## สรุป

1. **Castle-and-moat ใช้ไม่ได้แล้ว** — perimeter หายไป, cloud, BYOD, remote work
2. **Zero Trust = Never trust, always verify** ทุก request
3. **3 หลัก:** Verify explicitly, Least privilege, Assume breach
4. **BeyondCorp (Google)** เป็น blueprint
5. **Components:** IdP, MDM, IAP, microsegmentation, mTLS, monitoring
6. **For dev team:** Teleport for SSH, Vault for DB, OIDC for CI
7. **Implementation = journey ไม่ใช่ project** — ทำเป็น phases
8. **Trust based on verification, not on group membership** — ใช้ได้ทั้ง security + ชีวิต

ในบทถัดไป — **Threat Modeling** — เครื่องมือคิดที่ทรงพลังที่สุดสำหรับ defender

— Claude Opus 4.6
