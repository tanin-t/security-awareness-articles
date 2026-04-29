# บทความที่ 20: Threat Modeling — คิดแบบ Attacker เพื่อป้องกันแบบ Defender

**CyberSecurity Awareness Series — Part 8: Advanced Concepts**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของ Feature ที่ "ไม่มีใครคิดว่าจะพังได้"

เดือนกันยายน ปี 2018 — **Facebook** เปิดเผยว่า มีช่องโหว่ใน feature **"View As"** ที่ user สามารถดู profile ของตัวเองในมุมมองของคนอื่น ที่:

1. **ทำให้ผู้โจมตีได้ access tokens** ของ user อื่น
2. **กระทบบัญชีกว่า 50 ล้านบัญชี**
3. **ผู้โจมตีเข้าควบคุมบัญชี** ได้เต็มตัว
4. ใช้ tokens **ใน Facebook Login** บน apps อื่นได้ด้วย (Spotify, Pinterest, ฯลฯ)

ความเสียหาย:

- Facebook stock ตก **3%** ใน 1 วัน
- คดีฟ้อง class action รวม **$650 million**
- Reputation damage มหาศาล

ที่น่าตกใจ — **bug นี้อยู่มาตั้งแต่ปี 2017** (1 ปีก่อนเปิดเผย) — และเกิดจาก **interaction ของ 3 features** ที่ทำงานคนละทีม:

1. **"View As"** feature
2. **Video uploader**
3. **Birthday composer** (introduce ใน 2017)

แต่ละ feature ดูปลอดภัย — แต่เมื่อ chained กัน → leak access token

[อ่านรายละเอียด Facebook breach](https://www.theverge.com/2018/9/28/17914524/facebook-50-million-user-account-breach-data-bug)

> **ไม่มีใครคิดว่า 3 features นี้ chained กันได้ — ก่อนที่ผู้โจมตีจะเจอก่อน**

นี่คือเหตุผลที่ **threat modeling** สำคัญ — ก่อน deploy feature ใหม่ ต้อง **คิดเป็นผู้โจมตี** ก่อน เพื่อ identify pathways ที่ defender ไม่ทันเห็น

ในบทความนี้เราจะดู:

1. **Threat modeling คืออะไร**
2. **Frameworks** ที่นิยม (STRIDE, DREAD, Attack Trees)
3. **วิธีทำ** threat model สำหรับ feature ใหม่
4. **Integrate** เข้ากับ development process

---

## Threat Modeling คืออะไร

> **Threat modeling = การคิดล่วงหน้าว่า "อะไรอาจพังได้" และ "จะป้องกันยังไง"**

หลักการ — **ก่อน build** อะไร ให้:

1. **What are we building?** — เข้าใจระบบที่จะสร้าง
2. **What can go wrong?** — คิดว่าอะไรอาจพังได้
3. **What are we going to do about it?** — แผนป้องกัน
4. **Did we do a good job?** — verify ว่ามาตรการ work

นี่คือ **Adam Shostack's 4 Questions** — ที่ใช้กันทั่วโลกในปัจจุบัน

### ทำไมต้องทำ Threat Modeling

#### 1. ป้องกันก่อนเกิด

**ค่าใช้จ่ายในการแก้ vulnerability:**

| Stage | Relative Cost |
|---|---|
| Design phase | 1× |
| Development | 5× |
| Testing | 10× |
| Production | 100× |
| Post-breach | 1,000× |

→ เจอเร็ว = ถูกกว่า

#### 2. Force Conversation

Threat modeling ต้องคุยกัน — design + dev + security + ops มาคุยพร้อมกัน

→ Surface assumptions, hidden risks ตั้งแต่ก่อน build

#### 3. Document Decisions

เอกสาร threat model ทำให้:
- New team member เข้าใจ design rationale
- Audit ตรวจได้ว่า security ถูกพิจารณา
- Maintenance team ไม่พังโดยไม่ตั้งใจ

#### 4. Compliance

มาตรฐานหลายตัวต้องการ:
- **PCI-DSS 6.5** — secure development
- **ISO 27001** — risk assessment
- **NIST 800-53** — system security
- **SOC 2** — security review

---

## Adam Shostack's 4 Questions

นี่คือ framework ที่ง่ายและทรงพลัง — ใช้กับ feature ทุกตัว:

### Q1: What Are We Building?

#### Tools

- **Data Flow Diagram (DFD)** — แสดง data flow + trust boundaries
- **Architecture diagram**
- **Sequence diagram**

#### ตัวอย่าง DFD

```
┌─────────┐  HTTPS   ┌──────────┐   SQL    ┌──────────┐
│ Browser │ ───────→ │ Web App  │ ───────→ │ Database │
└─────────┘          └──────────┘          └──────────┘
                          │
                          │ HTTP
                          ↓
                     ┌──────────┐
                     │  3rd API │
                     └──────────┘

[Trust Boundaries]
  Browser ↔ Web App   ← เส้นกั้น 1
  Web App ↔ Database  ← เส้นกั้น 2 (สำคัญ!)
  Web App ↔ 3rd API   ← เส้นกั้น 3
```

#### องค์ประกอบของ DFD

- **External Entity** — outside system (browser, user, 3rd party)
- **Process** — ส่วนของระบบที่ process data (web app, lambda)
- **Data Store** — database, file system, cache
- **Data Flow** — arrows ระหว่างองค์ประกอบ
- **Trust Boundary** — เส้นที่ trust level เปลี่ยน

#### Trust Boundary คือจุดที่ Threat Model สนใจ

ทุก data ที่ **ข้าม trust boundary** = ต้อง verify

### Q2: What Can Go Wrong?

ใช้ framework ช่วยคิด:

#### STRIDE — by Microsoft (popular ที่สุด)

แต่ละตัวอักษร = 1 หมวดของ threat:

| Letter | Threat | Property Violated |
|---|---|---|
| **S** | Spoofing | Authentication |
| **T** | Tampering | Integrity |
| **R** | Repudiation | Non-repudiation |
| **I** | Information Disclosure | Confidentiality |
| **D** | Denial of Service | Availability |
| **E** | Elevation of Privilege | Authorization |

#### Per-Element STRIDE

แต่ละ element ใน DFD — เผชิญ threat ต่างกัน:

| Element | S | T | R | I | D | E |
|---|---|---|---|---|---|---|
| External Entity | ✓ |   | ✓ |   |   |   |
| Process | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Data Flow |   | ✓ |   | ✓ | ✓ |   |
| Data Store |   | ✓ | ? | ✓ | ✓ |   |

→ Process เผชิญทุก threat (most attack surface)

#### ตัวอย่าง STRIDE Application

**Login System:**

| Threat (STRIDE) | ตัวอย่าง |
|---|---|
| Spoofing | Phishing เพื่อขโมย credentials |
| Tampering | แก้ form input เพื่อ bypass validation |
| Repudiation | User ปฏิเสธว่าทำ transaction |
| Information Disclosure | Error message reveal valid usernames |
| Denial of Service | Brute force lockout legitimate users |
| Elevation | SQL injection ทำให้ได้ admin access |

#### DREAD — Risk Scoring

ใช้คำนวณ severity ของแต่ละ threat:

- **D** Damage — ความเสียหายถ้าโดน
- **R** Reproducibility — ทำซ้ำได้ง่ายแค่ไหน
- **E** Exploitability — โจมตียากแค่ไหน
- **A** Affected Users — กระทบกี่คน
- **D** Discoverability — หาช่องโหว่ง่ายแค่ไหน

แต่ละข้อให้คะแนน 1-10 → average = severity

```
Threat: SQL Injection in login form

Damage:           10  (full database access)
Reproducibility:  10  (works every time)
Exploitability:    8  (need basic SQL skills)
Affected Users:   10  (all users)
Discoverability:   7  (visible in source)
                ────
Average:          9.0  → Critical priority
```

DREAD ใช้น้อยลงในปัจจุบัน — เพราะ subjective + hard to be consistent

#### Attack Trees

Visualize attack paths:

```
GOAL: Steal user data
│
├── Path 1: SQL Injection
│   ├── Find unsanitized input
│   └── Craft injection payload
│
├── Path 2: Phishing
│   ├── Get user credentials
│   ├── Bypass 2FA
│   │   ├── SIM swap
│   │   └── Real-time MITM
│   └── Login as user
│
└── Path 3: Server Compromise
    ├── Find vulnerability
    └── RCE → access database
```

→ ระบุ paths ทั้งหมด + เลือก mitigation ที่กั้น path สำคัญ

#### MITRE ATT&CK Framework

Catalog ของ tactics + techniques ที่ผู้โจมตีจริงใช้ — แบ่งเป็น:

```
Tactics (เป้าหมาย):
  - Initial Access
  - Execution  
  - Persistence
  - Privilege Escalation
  - Defense Evasion
  - Credential Access
  - Discovery
  - Lateral Movement
  - Collection
  - Exfiltration
  - Impact

Techniques (วิธีทำ):
  - 600+ techniques organized under tactics
```

ใช้ ATT&CK เพื่อ:
- Map threats to known TTPs (Tactics, Techniques, Procedures)
- Identify gaps in defenses
- Prioritize controls

[attack.mitre.org](https://attack.mitre.org)

#### PASTA (Process for Attack Simulation and Threat Analysis)

7-stage methodology:

1. Define objectives
2. Define technical scope
3. Decompose application
4. Analyze threats
5. Vulnerability analysis
6. Attack analysis (simulate)
7. Risk + impact analysis

→ Heavyweight, ใช้กับ project ใหญ่

#### LINDDUN — Privacy Threat Modeling

เน้น privacy threats:

- **L** Linkability
- **I** Identifiability
- **N** Non-repudiation
- **D** Detectability
- **D** Disclosure of information
- **U** Unawareness
- **N** Non-compliance

ใช้กับ system ที่ handle PII (ที่กล่าวในบทที่ 18)

### Q3: What Are We Going to Do About It?

แต่ละ threat → mitigation:

#### Mitigation Strategies

| Approach | Description | Example |
|---|---|---|
| **Mitigate** | Reduce risk | Add input validation |
| **Eliminate** | Remove the feature | Don't store PII |
| **Transfer** | Pass risk to others | Use 3rd party auth |
| **Accept** | Acknowledge + document | Low-risk in dev env |

#### Common Mitigations

| Threat | Mitigation |
|---|---|
| Spoofing | Strong authentication, MFA |
| Tampering | Digital signatures, integrity checks |
| Repudiation | Audit logs, signed transactions |
| Information Disclosure | Encryption, access control |
| DoS | Rate limiting, scaling, CDN |
| Elevation | Principle of least privilege, input validation |

### Q4: Did We Do a Good Job?

Verify mitigations work:

- **Code review** — security-focused
- **Static analysis** (Semgrep, CodeQL)
- **Dynamic analysis** (DAST tools)
- **Penetration testing**
- **Bug bounty**
- **Red team exercises**
- **Threat model review** หลัง implementation

---

## STRIDE Walkthrough — Real Example

มาทำ threat model สำหรับ feature จริง:

### Feature: User Registration with Email Verification

```
[User Browser]
     ↓ HTTPS
[Frontend]
     ↓ HTTPS (REST API)
[Backend]
     ├─→ [Database] (user record)
     ├─→ [Email Service] (verification email)
     └─→ [Token Service] (verification token)

User flow:
1. Submit signup form (email, password)
2. Backend creates user (unverified)
3. Backend generates verification token
4. Backend sends verification email
5. User clicks link in email
6. Backend verifies token, marks user as verified
```

### STRIDE Analysis

#### Spoofing

**Threat:** Attacker registers with someone else's email

**Mitigation:**
- Email verification required before account active
- Don't allow login until verified
- Don't share account info until verified

**Threat:** Attacker fakes verification (phishing email)

**Mitigation:**
- Use unique verification token per request
- Token-only-known-to-server
- Don't show "verification token" in URL params on login

#### Tampering

**Threat:** Attacker modifies verification token in URL

**Mitigation:**
- Token = HMAC-SHA256 with server secret + expiration
- Verify signature server-side
- Don't trust token claims without verification

**Threat:** Attacker injects SQL via email field

**Mitigation:**
- Parameterized queries
- Email format validation
- ORM usage

#### Repudiation

**Threat:** User claims "I didn't sign up"

**Mitigation:**
- Audit log: signup time, IP, user agent, geolocation
- Verification email contains "If this wasn't you, click here to cancel"
- Send confirmation email after verification

#### Information Disclosure

**Threat:** Error message leaks "user already exists"

**Mitigation:**
- Generic message: "We've sent a verification email if your account doesn't exist"
- Same response for existing/new users
- Send actual email only if email is new

**Threat:** Verification token in email logs

**Mitigation:**
- Email service shouldn't log full email body
- Don't log token in app logs

#### Denial of Service

**Threat:** Attacker registers 1,000s of fake accounts

**Mitigation:**
- Rate limiting per IP (e.g., 5 signups/hour)
- CAPTCHA on signup form
- Disposable email blocking (basic)
- Monitor unusual signup patterns

**Threat:** Email flood (signing up legitimate emails to spam)

**Mitigation:**
- Rate limit per email address
- Verify email exists (DNS MX record)
- Honeypot field in form

#### Elevation of Privilege

**Threat:** Verification grants admin role accidentally

**Mitigation:**
- Default role = "user" (not admin)
- Role assignment separate from verification
- Admin role grant requires manual approval

**Threat:** Race condition: register + verify before validation completes

**Mitigation:**
- Atomic database transactions
- State machine: "unverified" → "verified" only via verification flow

### Output Document

```markdown
# Threat Model: User Registration

## What We're Building
[DFD attached]

## Threats Identified
1. Spoofing: Email impersonation
   - Severity: HIGH
   - Mitigation: Verification email + unique token
   
2. Tampering: Token modification
   - Severity: CRITICAL
   - Mitigation: HMAC signature + server-side verification
   
3. ... (etc)

## Decisions
- Don't accept disposable email addresses (block list)
- 1-hour expiration on verification tokens
- Rate limit: 5 signups per IP per hour

## Open Risks
- Email enumeration possible despite generic messages
   - Risk: LOW (timing attack)
   - Plan: monitor for unusual patterns

## Verification
- Code review by security team — DONE
- Penetration test — scheduled Q2
- Automated tests — covers happy path + error cases
```

---

## Threat Modeling for Common Patterns

### Pattern 1: REST API Endpoint

```
DFD:
[Client] → [API Gateway] → [App] → [Database]

Threats per element:
- Client (External Entity): Spoofing, Repudiation
- API Gateway (Process): All STRIDE
- App (Process): All STRIDE
- Database (Data Store): Tampering, Info Disclosure, DoS
- Network flows: Tampering, Info Disclosure

Common mitigations:
- HTTPS everywhere
- Authentication (JWT, OAuth)
- Authorization (RBAC, OWASP API Top 10)
- Rate limiting
- Input validation
- Parameterized queries
- Encryption at rest
```

### Pattern 2: File Upload

```
DFD:
[Client] → [App] → [File Storage (S3)]
                 → [Virus Scan]
                 → [Database (metadata)]

Threats:
- Spoofing: Upload malicious file disguised as image
- Tampering: Path traversal in filename (../../../etc/passwd)
- DoS: Upload huge files to fill disk
- Info Disclosure: Files accessible to wrong user
- Elevation: Server-side file execution (PHP, etc.)

Mitigations:
- Validate file type (magic bytes, not just extension)
- Sanitize filename (strip path components)
- File size limit
- Virus scan before processing
- Store outside web root or block execution
- Scoped access (per-user folders)
- Generate random filenames
```

### Pattern 3: Webhook Receiver

```
DFD:
[3rd Party] → HTTPS → [Webhook Endpoint] → [App]

Threats:
- Spoofing: Attacker sends fake webhook
- Replay: Attacker replays old webhook
- DoS: Flood with webhooks

Mitigations:
- Verify signature (HMAC with shared secret)
- Verify timestamp + nonce (replay protection)
- Rate limit per source IP
- Idempotency (handle duplicates)
- Allow-list source IPs if known
```

### Pattern 4: AI / LLM Integration

```
DFD:
[User] → [App] → [LLM API]
              → [Tool execution]

Threats (from บทที่ 17):
- Prompt injection (direct + indirect)
- Data leakage to provider
- Hallucinated outputs
- Tool misuse

Mitigations:
- Input length limit
- Pre-filter known injection patterns
- Tool sandboxing
- Output validation
- Human-in-loop for sensitive actions
- Audit logging
```

---

## Threat Modeling Process — Practical

### When to Do Threat Modeling

#### New Feature

ทุก feature ใหม่ที่:
- Touch sensitive data
- New attack surface
- Architectural change
- 3rd party integration

#### Major Refactor

เมื่อ change architecture significantly

#### Periodic Review

System ใหญ่ — review 1-2 ครั้ง/ปี

### Who Should Be Involved

#### Core Team

- **Product owner** — what are we building, why
- **Lead developer** — how it works
- **Security engineer** — threat expertise
- **DevOps / SRE** — operational perspective

#### Optional

- **UX designer** — flows + edge cases
- **Compliance officer** — regulatory
- **Data scientist** — if AI involved

### Format: Sticky Note Workshop

แบบที่นิยมที่สุด:

```
Setup:
  - 1 hour session
  - Whiteboard + sticky notes
  - Online: Miro, Figma

Steps:
  1. Draw DFD (15 min)
  2. Identify trust boundaries (5 min)
  3. STRIDE per element (30 min)
  4. Prioritize threats (5 min)
  5. Assign mitigations (5 min)
```

### Output Document

```markdown
# Threat Model: <Feature Name>

## Date: 2026-04-29
## Participants: Alice, Bob, Charlie, ...

## What We're Building
[DFD diagram link]
[Architecture description]

## Trust Boundaries
1. User ↔ Frontend
2. Frontend ↔ Backend
3. Backend ↔ Database
4. Backend ↔ 3rd Party API

## Threats & Mitigations

### High Priority
1. [Threat]: ... 
   Mitigation: ...
   Owner: ...
   Status: PLANNED

### Medium Priority
2. ...

### Low Priority
3. ...

## Accepted Risks
- [Risk]: ... 
  Reason: ...
  Compensating Control: ...

## Open Questions
- ...

## Review
- Implementation review: TBD
- Pen test: scheduled
```

---

## Tools for Threat Modeling

### Diagramming

- **Microsoft Threat Modeling Tool** (free) — STRIDE-based
- **OWASP Threat Dragon** (open source) — web-based
- **IriusRisk** — commercial, automation
- **Miro / FigJam** — collaborative whiteboarding
- **draw.io / diagrams.net** — free DFD

### Threat Library

- **MITRE ATT&CK Navigator** — visualize attacks
- **OWASP Top 10** — common web threats (chapter 14)
- **CWE** (Common Weakness Enumeration)

### Automation

- **Threagile** — code-based threat modeling (YAML → threats)
- **PyTM** — Python framework
- **OWASP Threat Modeling Manifesto** — principles

---

## Shift-Left Security

> **"Shift left" = ทำ security ตั้งแต่ต้นกระบวนการ ไม่ใช่ปลาย**

### Traditional Workflow

```
Plan → Design → Code → Test → Security Review → Deploy
                                ↑
                          Security เข้ามาที่นี่
                          ค่าใช้จ่ายแก้ไข = สูง
```

### Shift-Left

```
Plan → Design → Code → Test → Deploy
  ↑       ↑       ↑      ↑       ↑
Security ปนทุกขั้น
ค่าใช้จ่ายแก้ไข = ต่ำ
```

### Practical Shift-Left

#### Plan Phase

- **Threat modeling** (this article)
- Privacy review
- Compliance check

#### Design Phase

- **Security architecture review**
- Choose secure libraries
- Plan auth + authz strategy

#### Code Phase

- **Pre-commit hooks** (gitleaks, etc.)
- **IDE security plugins**
- **Pair programming** with security mind

#### Test Phase

- **Static analysis** (Semgrep, CodeQL)
- **Dependency scanning** (npm audit, Snyk)
- **Unit tests for security cases**

#### Deploy Phase

- **DAST** (Dynamic Application Security Testing)
- **Infrastructure scanning** (Checkov, tfsec)
- **Container scanning** (Trivy)
- **Penetration test** (periodic)

### Integrate Threat Modeling into Sprint

```
Sprint planning:
  - Review feature → quick threat model (30 min)
  - Identify security tasks
  - Add to sprint backlog

Sprint execution:
  - Implement security tasks
  - Code review with security lens

Sprint review:
  - Demo includes security testing
  - Update threat model based on changes

Retrospective:
  - "What new threats emerged?"
  - "Which mitigations worked?"
```

---

## Common Mistakes

### 1. Over-engineering

ทำ threat model แบบ super-detailed สำหรับทุก feature → exhaustion → ทีมเลิกทำ

→ **Right-size**: 30 min for small feature, 1-2 hours for new system

### 2. One-Time Activity

ทำครั้งเดียวตอนเริ่ม → ลืมไปเลย

→ **Living document** — update เมื่อ design เปลี่ยน

### 3. Security Team Only

Security ทำคนเดียว → dev ไม่ buy-in → mitigations ไม่ implement

→ **Collaborative**: dev + security + product ร่วมทำ

### 4. Document, Don't Implement

เขียน doc ละเอียด แต่ mitigations ไม่ถูกทำ

→ **Track มี ticket / task** สำหรับทุก mitigation

### 5. Ignore Operational Threats

มองแค่ application — ลืม:
- Insider threat
- Operational mistakes
- 3rd party risk

→ **Scope กว้าง** ต้องคิด

---

## Action Items

### วันนี้

- [ ] **List feature ปัจจุบัน** ที่ยังไม่มี threat model
- [ ] **Schedule threat modeling session** สำหรับ feature ถัดไป
- [ ] **อ่าน Adam Shostack's "Threat Modeling: Designing for Security"**

### สัปดาห์นี้

- [ ] **Try STRIDE** กับ feature เล็กๆ ก่อน (30 min)
- [ ] **สร้าง template** ของ threat model document
- [ ] **เพิ่ม security review step** ใน sprint process

### เดือนนี้

- [ ] **Threat model ใหญ่** สำหรับ critical system
- [ ] **Setup tool** (Threat Dragon / Microsoft TM Tool)
- [ ] **Train ทีม** เรื่อง STRIDE + DREAD
- [ ] **MITRE ATT&CK mapping** ของ threats

### สำหรับ Tech Lead / Architect

- [ ] **Threat modeling guidelines** ของบริษัท
- [ ] **Mandatory threat model** สำหรับ feature ที่ touch sensitive data
- [ ] **Quarterly review** ของ existing threat models
- [ ] **Build security champions** ใน every team

### สำหรับ Security Lead

- [ ] **Threat library** ของ common threats สำหรับ stack ของเรา
- [ ] **Automation** ของ threat modeling ที่ทำได้
- [ ] **Integration** เข้า CI/CD
- [ ] **Metrics** — how many features have threat models

---

## บทเรียนชีวิตจากบทความนี้

> **ก่อนตัดสินใจเรื่องสำคัญ ลองคิดว่า "อะไรอาจผิดพลาดได้" — เป็น mental model ที่ใช้ได้ทุกเรื่อง**

หลักการของ threat modeling = **Premortem thinking** — คิดล่วงหน้าว่า "ถ้าพังจะพังยังไง"

#### ในการตัดสินใจ Career

**ก่อน accept job ใหม่:**

- **What can go wrong?**
  - บริษัทล้ม
  - หัวหน้าใหม่ไม่ดี
  - งานไม่ตรงคาด
  - Politics
  - Visa issue
  - Compensation ไม่จริง

- **Mitigation:**
  - Reference check
  - Glassdoor reviews
  - Trial period
  - Contract review
  - Backup plan

#### ในการเงิน

**ก่อนลงทุน:**

- **What can go wrong?**
  - Market crash
  - Company default
  - Liquidity crisis
  - Regulatory change
  - Currency devaluation

- **Mitigation:**
  - Diversify
  - Emergency fund
  - Position sizing
  - Stop loss
  - Insurance

#### ในความสัมพันธ์

**ก่อน commitment:**

- **What can go wrong?**
  - Different values long-term
  - Communication breakdown
  - External pressure (family, friends)
  - Financial stress
  - Major life changes (kids, illness)

- **Mitigation:**
  - Open conversation early
  - Pre-marital counseling
  - Financial planning
  - Boundary setting
  - Couples therapy availability

#### ในธุรกิจ

**ก่อนเริ่ม startup:**

- **What can go wrong?**
  - Market doesn't want product
  - Competitor with more resources
  - Co-founder conflict
  - Cash runs out
  - Key employee leaves
  - Customer concentration risk

- **Mitigation:**
  - Customer validation first
  - Co-founder agreement
  - Runway buffer
  - Diversify customers
  - Documentation

> **คนที่ผ่านทางได้ — ไม่ใช่คนที่ "หวังให้ทุกอย่างดีไป" แต่เป็นคนที่ "เตรียมไว้สำหรับสิ่งที่ผิดพลาด"**

หลักการ premortem ของ Gary Klein:

```
Imagine: 1 year from now, the project is a complete failure.
Question: What went wrong?
Action: Address those potential failures NOW.
```

ใช้ได้ทุกการตัดสินใจสำคัญ:

1. **Visualize failure** ในรายละเอียด
2. **Brainstorm causes** ของ failure
3. **Categorize** (high vs low likelihood, high vs low impact)
4. **Mitigate** สิ่งที่สำคัญที่สุด
5. **Accept + monitor** สิ่งที่ acceptable

> **คนที่คิดเป็น attacker — มี advantage ในการป้องกัน**  
> **คนที่คิดเป็น failure-finder — มี advantage ในชีวิต**

> **Optimist พูดว่า "ทุกอย่างจะดี"**  
> **Pessimist พูดว่า "ทุกอย่างจะแย่"**  
> **Realist พูดว่า "ลองดูว่าอะไรอาจผิด แล้วเตรียมตัว"**

ในชีวิตที่ uncertain — realist ที่เตรียมตัวคือคนที่อยู่รอด

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไป — **Incident Response** — เมื่อสิ่งที่ "ไม่ควรเกิด" เกิดขึ้น ทำยังไง

---

## อภิธานศัพท์ (Glossary)

- **Threat Modeling:** กระบวนการคิดล่วงหน้าว่าระบบอาจถูกโจมตียังไง
- **STRIDE:** Microsoft framework — Spoofing, Tampering, Repudiation, Info Disclosure, DoS, Elevation
- **DREAD:** Risk scoring — Damage, Reproducibility, Exploitability, Affected, Discoverability
- **Attack Tree:** Tree diagram ของ attack paths
- **MITRE ATT&CK:** Catalog ของ tactics + techniques ของ real attackers
- **PASTA:** 7-stage threat modeling methodology
- **LINDDUN:** Privacy-focused threat modeling framework
- **DFD (Data Flow Diagram):** แผนภาพแสดง data flow + trust boundaries
- **Trust Boundary:** เส้นที่ trust level เปลี่ยน
- **Premortem:** เทคนิคคิดว่า project fail แล้วถามว่าทำไม
- **Shift-Left:** ทำ security ตั้งแต่ต้นกระบวนการ
- **Adam Shostack's 4 Questions:** What building? What can go wrong? What to do? Did we do well?
- **TTPs (Tactics, Techniques, Procedures):** ของผู้โจมตี
- **External Entity:** องค์ประกอบนอกระบบ (user, 3rd party)
- **Process:** ส่วนของระบบที่ process data
- **Data Store:** database, file system, cache
- **Data Flow:** การไหลของ data ระหว่างองค์ประกอบ
- **Mitigation:** มาตรการลด risk
- **CWE (Common Weakness Enumeration):** ฐานข้อมูล software weakness

---

## สรุป

1. **Threat modeling = คิดล่วงหน้าว่าจะพังยังไง** — ป้องกันก่อนเกิด
2. **Adam Shostack's 4 Questions** — what building, what wrong, what do, did well
3. **STRIDE** เป็น framework ที่ใช้ง่ายและทรงพลัง
4. **DREAD** สำหรับ scoring (subjective)
5. **Attack trees** สำหรับ attack paths
6. **MITRE ATT&CK** สำหรับ real-world TTPs
7. **DFD + trust boundaries** เป็นพื้นฐาน
8. **Shift-left** — security ตั้งแต่ต้น = ถูกกว่ามาก
9. **Living document** — update เสมอ
10. **Premortem thinking** ใช้ได้ในชีวิต — ก่อนตัดสินใจสำคัญ

ในบทถัดไป — **Incident Response** — เมื่อทุกอย่างพังจริง ทำอะไรต่อ

— Claude Opus 4.6
