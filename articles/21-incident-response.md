# บทความที่ 21: Incident Response — เมื่อโดน Hack แล้ว ทำอะไรต่อ

**CyberSecurity Awareness Series — Part 8: Advanced Concepts**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของบริษัทที่ "ตอบสนองได้ดี" vs บริษัทที่ "ตอบสนองพัง"

เดือนเดียวกันในปี 2017 — มีสองบริษัทใหญ่โดน data breach ใกล้เคียงกัน — แต่**ผลลัพธ์ต่างกันมาก** เพราะวิธีตอบสนอง

### Equifax — Incident Response ที่ "พัง"

มีนาคม 2017 — **Equifax** (1 ใน 3 credit bureaus ใหญ่ของอเมริกา) ถูกแฮกผ่าน:

- Apache Struts vulnerability (CVE-2017-5638) ที่มี patch ออกแล้ว 2 เดือน
- **Equifax ไม่ patch**
- ผู้โจมตีเข้าระบบและอยู่นาน **76 วัน**
- ขโมยข้อมูล **147 ล้านคน** — SSN, DOB, address, driver's license

**สิ่งที่ Equifax ตอบสนองพัง:**

1. **ค้นพบช้า** — ใช้เวลา 6 สัปดาห์กว่าจะรู้
2. **ประกาศช้า** — รู้กรกฎาคม ประกาศกันยายน (40+ วัน)
3. **ผู้บริหารขาย stock** ก่อนประกาศ (insider trading)
4. **เว็บ "ตรวจสอบ"** ที่ทำขึ้นมา — ไม่ทำงาน + ดูเหมือน phishing
5. **Twitter ทำผิด** — แชร์ลิงก์ phishing แทนของจริง
6. **CEO + CIO + CISO ลาออก** ภายในไม่กี่สัปดาห์

ผลลัพธ์:
- **$1.4 billion** ใน fines + settlements
- Stock ตก **35%** ใน 1 สัปดาห์
- Reputation damage ตีค่าไม่ได้
- Class action จาก 100M+ users

[อ่านรายละเอียด Equifax breach](https://en.wikipedia.org/wiki/2017_Equifax_data_breach)

### Cloudflare — Incident Response ที่ "ดี"

กุมภาพันธ์ 2017 — **Cloudflare** ค้นพบ bug ที่เรียกว่า **"Cloudbleed"** — buffer overflow ใน HTML parser ที่ leak random memory:

- Memory ที่รั่วอาจมี HTTPS keys, cookies, passwords ของ Cloudflare customers
- **Cloudflare เป็น CDN ของเว็บ 5+ ล้านแห่ง** — กระทบกว้างมาก

**สิ่งที่ Cloudflare ตอบสนองดี:**

1. **Project Zero ของ Google แจ้ง** วันที่ 17 ก.พ. 17:53 UTC
2. **44 นาที** ต่อมา → Cloudflare มี initial mitigation
3. **3 ชั่วโมง** ต่อมา → root cause identified + permanent fix in PR
4. **Public disclosure** วันที่ 23 ก.พ. — เขียน blog post ที่:
   - บอก timeline ละเอียด
   - บอก scope ที่กระทบ
   - บอกสิ่งที่ทำเพื่อ mitigate
   - บอกสิ่งที่จะป้องกันอนาคต
5. **Worked with search engines** ลบ cached content ที่ leaked

ผลลัพธ์:
- **No customer breach reported**
- Reputation **เพิ่มขึ้น** เพราะ transparency
- **Industry praise** ในการตอบสนอง

[อ่าน Cloudflare's incident report](https://blog.cloudflare.com/incident-report-on-memory-leak-caused-by-cloudflare-parser-bug/)

> **Bug + Bad Response = Disaster**  
> **Bug + Good Response = Forgivable, Even Respected**

ในชีวิตจริง — **ไม่มีบริษัทไหนไม่เคย incident** — ความต่างคือ **เตรียมตัวยังไง** + **ตอบสนองยังไง**

ในบทความนี้เราจะดู:

1. **ทำไมต้องมี IR plan** — ตื่นตระหนกคือศัตรูตัวจริง
2. **5 ขั้นตอน** ของ IR
3. **Sub-plans** สำหรับ dev team
4. **Tabletop exercises** — ซ้อมเหมือนซ้อมหนีไฟ
5. **Post-mortem** ที่ build culture ไม่ใช่ blame

---

## ทำไมต้องมี IR Plan — ตื่นตระหนกคือศัตรูตัวจริง

### Without Plan: Panic

```
14:00 — Alert: production down
14:01 — Engineer A: "What's happening?"
14:02 — Engineer B: "I don't know. Let me check."
14:05 — "I think someone hacked us!"
14:10 — Manager: "Call the CEO!"
14:11 — CEO: "What?! Get IT NOW!"
14:30 — Still no clear action
14:45 — CTO joins, asks for status, no one knows
15:00 — Customer support flooded with questions
15:15 — Marketing posts random tweet: "We're investigating"
16:00 — Twitter explodes with speculation
17:00 — News article goes live
18:00 — Stock drops
```

→ **Chaos** — ทุกคนทำอะไรไม่รู้ + เสียเวลา + สื่อสารพัง

### With Plan: Calm Execution

```
14:00 — Alert: production down
14:01 — On-call engineer triages
14:02 — Severity assessed: SEV-1
14:03 — IR slack channel auto-created, key people pinged
14:05 — IR Commander (predefined) takes lead
14:08 — Roles assigned: containment, comms, scribe
14:15 — First containment action taken
14:20 — Customer notification drafted
14:25 — Stakeholders updated (board, legal, PR ready)
15:00 — Issue contained
15:30 — Investigation underway
17:00 — Public statement issued
18:00 — Recovery in progress
20:00 — Service restored
Tomorrow — Post-mortem scheduled
```

→ **Order** — แต่ละคนรู้บทบาท + การทำงานแบบ choreographed

### ประโยชน์ของ IR Plan

1. **ลด response time** — รู้ทำอะไรทันที
2. **Limit damage** — contain ก่อน spread
3. **Avoid mistakes** — ไม่ตัดสินใจตอน panic
4. **Compliance** — กฎหมายต้องการ
5. **Insurance** — cyber insurance ต้องการ
6. **Customer trust** — แสดงว่ามือมาตรฐาน
7. **Legal protection** — แสดงว่าทำตาม "reasonable care"

---

## NIST IR Lifecycle (4 Phases)

NIST SP 800-61 framework — มาตรฐานที่ใช้กันทั่วโลก:

```
┌────────────────────┐
│  1. Preparation    │ ← Before
└────────────────────┘
          ↓
┌────────────────────┐
│  2. Detection &    │ ← During (continuous)
│     Analysis       │
└────────────────────┘
          ↓
┌────────────────────┐
│  3. Containment,   │ ← During (act)
│     Eradication,   │
│     Recovery       │
└────────────────────┘
          ↓
┌────────────────────┐
│  4. Post-Incident  │ ← After
│     Activity       │
└────────────────────┘
```

ใน practical world เรามักเห็น 5-step version:

**Identify → Contain → Eradicate → Recover → Lessons Learned**

มาดูรายละเอียดทีละขั้น

---

## Phase 1: Preparation — สิ่งที่ต้องทำก่อนเกิด

นี่คือส่วนที่ **สำคัญที่สุด** — สิ่งที่ทำได้ก่อนเกิดเรื่อง

### Documentation

#### Incident Response Plan

```markdown
# IR Plan v1.0

## Scope
What incidents this plan covers (security, outage, data breach, etc.)

## Severity Levels
- SEV-1: Critical — production down, data breach, major risk
- SEV-2: High — significant impact, but contained
- SEV-3: Medium — limited impact
- SEV-4: Low — minor issue

## Roles
- Incident Commander (IC)
- Tech Lead
- Communications Lead
- Scribe
- Legal Counsel (on-call)

## Phases
1. Detection: how alerts come in
2. Triage: severity assessment
3. Response: per-severity playbook
4. Recovery: restore services
5. Post-mortem: schedule + format

## Communication
- Internal: #incidents Slack channel
- External: PR/Comms team
- Legal: lawyer@company.com
- Customers: status page + email

## Contacts
[Phone numbers for key people, on-call schedule]
```

#### Playbooks (Per Incident Type)

```markdown
# Playbook: Suspected Data Breach

## Detection Indicators
- Unusual data exfiltration patterns
- Unauthorized access alerts
- Customer reports
- Dark web monitoring alerts
- Anonymous tips

## Immediate Actions (first 15 minutes)
1. Don't panic
2. Page IC + tech lead
3. Open IR channel
4. Identify scope:
   - What data?
   - How much?
   - From when?
5. Don't shutdown systems unless safe
6. Don't delete logs

## Evidence Preservation
- Snapshot affected systems
- Capture memory if possible
- Preserve network logs
- Document all actions

## Containment Options
[List specific commands/actions for each scenario]

## Communication Templates
[Pre-drafted messages for legal/customers/PR]

## Legal Notifications Required
- PDPA: 72 hours to PDPC
- GDPR: 72 hours to DPA
- US states: varies
- Customers: per regulation
```

#### Runbooks

Step-by-step technical procedures:
- "How to revoke all AWS access keys"
- "How to rotate database passwords"
- "How to take EC2 forensic image"
- "How to engage external IR firm"

### Tools & Access

#### IR Tools

- **SIEM** (Splunk, Elastic, Sumo Logic) — log aggregation + search
- **EDR** (CrowdStrike, SentinelOne) — endpoint visibility
- **NDR** (Network Detection) — network visibility
- **Forensic tools** — for evidence collection
- **Communication** — separate channel that doesn't depend on company infra (in case it's compromised)

#### Pre-staged Access

- IR team has emergency access (break-glass account)
- Tested quarterly
- Rotated immediately after use

### Team & Training

#### IR Team Composition

- **IC (Incident Commander)** — leads, decides, doesn't do tech work
- **Tech Lead** — technical decisions
- **Comms Lead** — handles communication
- **Scribe** — documents everything
- **Legal Counsel** — on-call
- **PR/Communications** — for public-facing
- **Engineering** — depends on incident
- **Executive Sponsor** — for major incidents

#### External Resources

- **IR Firm Retainer** — Mandiant, CrowdStrike, KPMG, etc.
- **Cyber Insurance** — pre-approved coverage
- **External Legal** — specialized in cyber
- **PR Firm** — crisis communications

→ Have these on **retainer** before need — not when crisis hits

### Training

#### Tabletop Exercises (เราจะลึกในส่วนถัดไป)

#### Live Drills

Simulate real incident with red team

#### Continuous Training

- Quarterly IR training
- Annual tabletop
- New hire IR onboarding

---

## Phase 2: Detection & Analysis — ดูได้ทันไหม

### Sources of Detection

#### Internal Detection

- **SIEM alerts** — log analysis
- **EDR alerts** — endpoint anomaly
- **Application monitoring** — error spikes, perf issues
- **Network monitoring** — unusual traffic
- **User reports** — IT helpdesk tickets

#### External Detection

- **Customer reports** — "Why is my data on dark web?"
- **Bug bounty** — researcher reports
- **Law enforcement** — FBI warnings
- **Vendor notification** — supply chain compromise
- **Press / social media** — sometimes media finds first

### Initial Triage

```python
# Triage checklist
def triage_alert(alert):
    questions = [
        "Is this a true positive?",  # not just noise
        "What's the severity?",       # SEV-1/2/3/4
        "What's the scope?",          # 1 system or many
        "Is sensitive data involved?", # PII, PHI, financial
        "Is it actively spreading?",   # contained or active
        "Are there indicators of compromise (IOCs)?",
        "Is the attacker still active?"
    ]
    # Each answer affects severity + response
```

### Incident vs Event

- **Event** — anything observable (login, error)
- **Alert** — event that needs review
- **Incident** — confirmed security/operational issue

→ ไม่ใช่ทุก alert = incident

### Common Mistake: Alert Fatigue

ถ้า alerts มากเกินไป → ทีมเริ่ม ignore → real incident หลุดออก

**Solution:**
- Tune alerts (false positive rate < 5%)
- Severity-based routing
- Auto-close low-severity
- Periodic review

---

## Phase 3: Containment — หยุดการแพร่กระจาย

### Short-term Containment

**Stop the bleeding** — ทำเร็วที่สุดเพื่อ limit damage:

#### Network Containment

```bash
# Isolate affected hosts
iptables -A INPUT -s <affected_ip> -j DROP
iptables -A OUTPUT -d <attacker_ip> -j DROP

# Disable network port
network-admin disable port <port>

# Quarantine in EDR
crowdstrike-cli isolate <hostname>
```

#### Account Containment

```bash
# Disable user
aws iam update-user --user-name compromised-user --status Inactive
aws iam delete-access-key --access-key-id <key>
aws iam attach-user-policy --user-name compromised-user --policy-arn DenyAll

# Revoke sessions
aws cognito-idp admin-user-global-sign-out --username compromised-user

# Force MFA
saml-cli enable-mfa-required --user compromised-user
```

#### Service Containment

```bash
# Disable affected service
kubectl scale deployment compromised-service --replicas=0

# Block from registry
gcloud artifacts repositories delete compromised-repo

# Revoke all API tokens
api-cli revoke-all-tokens --service compromised-service
```

### Long-term Containment

หลัง emergency stop — มาตรการที่ stable กว่า:

- **Deploy patches** ที่แก้ vulnerability
- **Add monitoring** ที่ detect future attempts
- **Update access policies**
- **Rebuild** affected systems clean

### Don't Tip Off the Attacker

ถ้า attacker ยัง active:
- **อย่ารีบ** kick them out — อาจไม่ทันเก็บ evidence
- **Watch quietly** เพื่อ understand scope
- **Coordinate** กับ law enforcement
- **เผื่อมี backdoor** ที่ยังไม่เจอ

→ บางครั้ง — observe ก่อน eradicate

---

## Phase 4: Eradication — ขจัดให้หมด

### Remove Malicious Artifacts

- Malware
- Backdoors
- Web shells
- Persistence mechanisms (cron jobs, systemd, registry)
- Compromised accounts
- Modified configurations

### Verify Clean State

อย่าเชื่อว่า "ลบแล้ว" — verify:

```bash
# Compare to known-good baseline
diff /etc/passwd /backup/etc/passwd
diff /etc/shadow /backup/etc/shadow
diff -r /etc/systemd/ /backup/etc/systemd/

# Hash binaries
sha256sum /usr/bin/* > /tmp/binaries.sha256
diff /tmp/binaries.sha256 /baseline/binaries.sha256

# Network listeners
netstat -tulpn  # Unexpected listeners?

# Running processes
ps auxf  # Unexpected processes?

# Cron jobs
crontab -l
ls -la /etc/cron.*

# Recently modified files
find / -mtime -7 -type f
```

### Sometimes — Rebuild from Scratch

For critical systems — **don't trust** compromised infrastructure:

- Rebuild from known-good images
- Don't restore from backup ที่อาจ tainted
- Reissue all certificates
- Rotate all secrets

---

## Phase 5: Recovery — กลับสู่ปกติ

### Gradual Restoration

```
Stage 1: Internal services back online
Stage 2: Limited customer access (canary)
Stage 3: Full customer access
Stage 4: Normal operations
```

ระหว่างทาง — **enhanced monitoring** เผื่อ attacker กลับมา

### Verification

ก่อนประกาศ "all clear":

- [ ] All malicious artifacts removed
- [ ] Vulnerabilities patched
- [ ] Credentials rotated
- [ ] Logs analyzed for any missed activity
- [ ] Monitoring enhanced
- [ ] Incident IOCs added to detection

### Communication

#### Internal

```
TO: All employees
SUBJECT: Update on Recent Incident

We've successfully resolved the incident. Here's what happened and what we're doing:

WHAT: [brief description]
IMPACT: [scope]
ACTION TAKEN: [containment/eradication]
NEXT STEPS: [improvements]
QUESTIONS: contact security@company.com

Thanks for your patience.
```

#### External

ขึ้นกับ regulation + risk:

- **PDPA**: notify PDPC within 72 hours if certain criteria met
- **GDPR**: notify DPA within 72 hours
- **Customer notification**: per breach severity
- **Press release**: if public knowledge already

#### Don't Speculate

- Stick to facts
- "Investigation ongoing" is OK
- Don't promise things that may not be true
- Don't blame employees publicly

---

## Phase 6: Lessons Learned — สำคัญที่สุดที่หลายบริษัทข้าม

### Post-Mortem (Blameless)

ภายใน 1-2 สัปดาห์หลัง incident:

#### Format

```markdown
# Post-Mortem: <Incident>

## Date
2026-04-29

## Severity
SEV-1

## Summary
[1-2 sentences]

## Timeline
- HH:MM — Event
- HH:MM — Detection
- HH:MM — IC engaged
- HH:MM — Containment
- HH:MM — Recovery
- HH:MM — Resolved

## Impact
- Customers affected: X
- Revenue lost: $Y
- Data exposed: ...
- Downtime: Z hours

## Root Cause
[Technical + organizational]

## What Went Well
- ...

## What Went Wrong
- ...

## Action Items
- [ ] Fix root cause (Owner: A, Due: 2026-05-15)
- [ ] Improve monitoring (Owner: B, Due: 2026-06-01)
- [ ] Update runbook (Owner: C, Due: 2026-05-08)
```

### Blameless Culture

> **Hindsight bias makes everyone look stupid. Don't blame humans — fix systems.**

#### Bad Post-Mortem

> "John was the on-call engineer. He failed to respond to the alert in time. He should be more careful."

→ John defends, hides, doesn't volunteer to be on-call again

#### Good Post-Mortem

> "Alert was sent at 14:02. On-call engineer was paged. Acknowledgement came at 14:23 (target: 5 min). Investigation revealed:
> - Alert appeared in #monitoring instead of paging system (alert routing bug)
> - Phone wasn't on (engineer's first time on-call)
> - On-call onboarding doc didn't mention phone-must-be-on
>
> Action items:
> - Fix alert routing (Owner: SRE)
> - Update on-call onboarding (Owner: PM)
> - Add automated 'phone test' before go-live (Owner: SRE)
> - Pair new on-call with experienced for first month (Owner: Manager)"

→ Improvements ที่ระบบ ไม่ใช่ blame คน

### Etsy's Just Culture

[Etsy's blog post](https://www.etsy.com/codeascraft/blameless-postmortems) เป็น classic — บอกว่า:

> *"We want to know what factors made it possible for someone to make a mistake. The 'stupidity' of the action is in our system, not in the person."*

### Update Documentation

หลัง post-mortem:

- Update playbooks
- Update runbooks
- Update threat model (จากบทที่ 20)
- Update training materials

### Track Action Items

จาก post-mortem → Jira / Linear / GitHub issues

→ Don't let action items rot — review monthly

---

## Sub-Plans for Dev Team

### Compromised Credentials

```bash
# IF developer's GitHub token leaked

1. Immediately rotate
   gh auth login --scopes repo,workflow

2. Audit usage
   gh api /user/access-tokens

3. Search for token in git history
   gitleaks detect --source . --log-opts="--all"

4. Rewrite history if needed (or just rotate everywhere)

5. Audit recent commits/PRs from that user

6. Notify affected repo owners

7. Update CI secrets
```

### Compromised Production Server

```bash
# IF EC2 instance compromised

1. Isolate (security group → deny all egress)
   aws ec2 modify-instance-attribute --instance-id i-xxx \
     --groups sg-deny-all

2. Snapshot for forensics
   aws ec2 create-snapshot --volume-id vol-xxx \
     --description "incident-snapshot"

3. Don't terminate yet (preserve evidence)

4. Identify scope
   - What data was on this instance?
   - What did it have access to?
   - When was last login?

5. Rotate credentials
   - IAM roles attached to instance
   - Database passwords accessible from instance
   - API keys

6. Investigate
   - CloudTrail for instance API calls
   - VPC Flow Logs for traffic
   - System logs

7. Once investigation done — terminate + rebuild
```

### Suspected Insider Threat

```bash
# IF suspect employee misuse

1. Don't tip off the person yet
2. Engage HR + Legal
3. Preserve evidence:
   - Email
   - File access logs
   - Cloud activity
4. Plan coordinated response:
   - Disable access
   - Recover company assets
   - Legal proceedings if needed
```

### Ransomware

ที่กล่าวในบทที่ 10 — refer to that

---

## Tabletop Exercises — ซ้อมเหมือนซ้อมหนีไฟ

### What is a Tabletop

**Tabletop exercise** = simulated incident on paper — ไม่ touch real systems

ทีมนั่งคุยกัน:
- "Imagine: We just got a call from FBI saying our customer data is on dark web"
- "What do we do?"
- "Who do we call?"
- "Who decides what?"

### Why Tabletop Works

1. **Find gaps in plan** before real incident
2. **Practice coordination** — different teams must work together
3. **Build muscle memory** — actions feel familiar in real crisis
4. **Identify dependencies** — "wait, who has the legal contact?"
5. **Test communications** — Can we reach key people fast enough?

### Format (90-minute exercise)

```
0:00-0:10  Setup
  - Facilitator introduces scenario
  - Participants briefed on rules

0:10-0:30  Initial Phase
  - "It's 14:00 on Tuesday"
  - "Customer support gets call: customer's CC charged $0.01 from us, then $5,000 from somewhere else"
  - "What do you do?"

0:30-0:50  Escalation
  - Inject new info
  - "It's been 1 hour. Twitter starts trending."

0:50-1:10  Resolution
  - Teams work toward containment + comms

1:10-1:30  Hot wash (debrief)
  - What went well
  - What went poorly
  - Action items
```

### Scenarios to Practice

#### Easy / Common

- Phishing email reported
- Single employee laptop infected
- API rate limit attack

#### Medium

- Misconfigured S3 bucket discovered
- Insider threat suspicion
- Vendor breach affecting us

#### Hard / Catastrophic

- Ransomware attack
- Data breach (millions of records)
- Supply chain attack
- DDoS during peak business
- Executive compromised
- Cloud provider outage

### Cadence

- **Quarterly**: small scenarios
- **Annually**: large/complex
- **After major changes**: scenario adapted to new architecture

---

## IR Maturity Levels

### Level 0: Ad-hoc

- No plan
- Whoever is around responds
- Mistakes common

### Level 1: Reactive

- Basic plan exists
- Used during incidents
- No drills

### Level 2: Defined

- Documented plan
- Regular updates
- Some drills

### Level 3: Managed

- Metrics tracked (MTTD, MTTR)
- Regular drills
- Cross-team training
- External validation

### Level 4: Optimized

- Continuous improvement
- Threat intelligence integrated
- Automated response (SOAR)
- Security operations center (SOC)

→ Most companies = Level 1-2  
→ Mature security organizations = Level 3-4

### Key Metrics

#### MTTD (Mean Time to Detect)

How long from breach to detection?

- Industry average: **204 days** (IBM Cost of Breach 2024) — too long!
- Mature orgs: <24 hours

#### MTTC (Mean Time to Contain)

From detection to containment?

- Industry average: 73 days
- Target: <24 hours for SEV-1

#### MTTR (Mean Time to Recover)

From containment to full recovery?

- Depends on scope

#### Cost of Breach

- Average: $4.88M (2024)
- With IR plan: $2.66M
- → IR plan saves $2M+ per incident

---

## Action Items

### วันนี้

- [ ] **เช็คว่ามี IR plan** ไหม — written, not in someone's head
- [ ] **List on-call contacts** — phone numbers, alternates
- [ ] **Test break-glass access** ในเครื่อง production

### สัปดาห์นี้

- [ ] **Schedule tabletop exercise** ใน 2 สัปดาห์
- [ ] **Update runbooks** — credentials rotation, server isolation
- [ ] **Identify external resources** — IR firm retainer, legal counsel

### เดือนนี้

- [ ] **Run first tabletop** with leadership
- [ ] **Setup metrics tracking** — MTTD, MTTC, MTTR
- [ ] **Review past incidents** — apply lessons
- [ ] **Cyber insurance review** — claims process clear

### สำหรับ CISO / Security Lead

- [ ] **IR plan document** ที่ board signed off
- [ ] **Quarterly tabletop schedule**
- [ ] **External relationships** — IR firm, law enforcement, regulators
- [ ] **Insurance** with adequate coverage
- [ ] **24/7 on-call** for SEV-1

### สำหรับ Engineering Manager

- [ ] **On-call training** ที่จริงจัง
- [ ] **Blameless culture** — explicit message
- [ ] **Post-mortem template** standard
- [ ] **Action item tracking** — never let them rot

### สำหรับ Executive

- [ ] **Approve IR plan**
- [ ] **Resource allocation** for IR
- [ ] **Crisis communication** training
- [ ] **Board notification** procedure

---

## บทเรียนชีวิตจากบทความนี้

> **เตรียมแผนสำหรับสิ่งที่ไม่อยากให้เกิด — วิกฤตไม่น่ากลัวเท่าการไม่มีแผนรับมือ**

ในชีวิตจริง — ทุกคน "ไม่อยาก" ให้เกิดเรื่องแย่ๆ:

#### ไฟไหม้บ้าน

- ไม่มีใครอยากให้เกิด
- แต่บ้านที่มีถังดับเพลิง + เครื่องตรวจควัน + แผนหนี = รอด
- บ้านที่ไม่มี = หายนะ

#### อุบัติเหตุรถยนต์

- ไม่มีใครอยากให้เกิด
- แต่รถที่มีประกัน + ต่อภาษี + คนขับมีบัตรเครดิตฉุกเฉิน = ฟื้นได้
- รถที่ไม่มีอะไร = ติดหนี้ยาว

#### สูญเสียคนใกล้ตัว

- ไม่มีใครอยากให้เกิด
- แต่ครอบครัวที่มีพินัยกรรม + ประกันชีวิต + กระบวนการพูดคุย = adjust ได้
- ครอบครัวที่ไม่มี = ทะเลาะมรดก + ปัญหาการเงิน

#### ตกงาน

- ไม่มีใครอยากให้เกิด
- คนที่มี emergency fund 6 เดือน + network + skills update = หางานใหม่ได้
- คนที่ไม่มี = วิกฤตการเงิน

> **คนที่ "เตรียมตัวสำหรับสิ่งที่ไม่อยากเกิด" — ไม่ใช่ pessimist แต่เป็น realist ที่ smart**

#### Mental Model: "What Would I Do If..."

ฝึกตั้งคำถามนี้กับชีวิต:

- "ถ้าตกงานพรุ่งนี้ ฉันจะทำอะไร?"
- "ถ้าหัวหน้าออก ฉันจะติดต่อใคร?"
- "ถ้าแฟนเลิก ฉันมีที่อยู่ไหม?"
- "ถ้ามือถือหาย วันนี้ ฉันยังเข้าบัญชีได้ไหม?"
- "ถ้าพ่อแม่ป่วยกะทันหัน เงินสำรองพอไหม?"

ถ้าตอบไม่ได้ — **เตรียม**

> **Crisis ไม่ใช่ถ้า แต่เป็น "เมื่อไหร่"**  
> **คนที่อยู่รอด ไม่ใช่คนที่ "ไม่เคยเจอวิกฤต" แต่คือคนที่ "เตรียมตัวก่อน"**

#### Stoic Wisdom

นักปรัชญา Stoic ของโรมัน (Seneca, Marcus Aurelius) มี practice ชื่อ **premeditatio malorum** — "premeditation of evils":

```
ทุกเช้า ก่อนเริ่มวัน — visualize:
- "วันนี้อาจจะมีคนใจร้ายกับฉัน"
- "วันนี้อาจจะมีอะไรพังที่ทำงาน"
- "วันนี้อาจจะมีข่าวร้าย"

แล้วถาม:
- "ถ้าเกิดขึ้น ฉันจะตอบสนองยังไง?"
- "ฉันยังเป็นคนดีต่อไปได้ไหม?"
- "ฉันจะ recover ยังไง?"
```

→ พอเกิดจริง → ไม่ shock → response ที่มาตรฐาน

ในเรื่อง software:

- **Production มีแน่ down** — มี SLA, on-call, IR plan
- **Code มีแน่ bug** — มี test, monitoring, rollback
- **Customer มีแน่ complain** — มี process, response template
- **Data มีแน่ loss** — มี backup (บทที่ 11)

> **คนที่เตรียมตัวสำหรับวิกฤต — มี calmness ในวันที่คนอื่น panic**

นั่นคือ superpower ที่ใช้ได้ทั้งในงานและในชีวิต

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไป Part 9 — **Security Culture** — เพราะ security ไม่ใช่เรื่องของคนๆ เดียว แต่ของทุกคนในทีม

---

## อภิธานศัพท์ (Glossary)

- **Incident:** confirmed security/operational issue
- **Event:** observable activity (log entry)
- **Alert:** event that needs review
- **IR (Incident Response):** กระบวนการตอบสนองต่อ incident
- **IRP (Incident Response Plan):** documented plan
- **IC (Incident Commander):** ผู้นำ IR ที่ตัดสินใจ
- **Playbook:** step-by-step guide สำหรับ scenario เฉพาะ
- **Runbook:** technical procedure
- **NIST 800-61:** US framework สำหรับ IR
- **MTTD (Mean Time to Detect):** เวลาเฉลี่ย detect
- **MTTC (Mean Time to Contain):** เวลาเฉลี่ย contain
- **MTTR (Mean Time to Recover):** เวลาเฉลี่ย recover
- **SEV (Severity):** ระดับความรุนแรง (SEV-1 = critical)
- **IOC (Indicator of Compromise):** หลักฐานของ compromise
- **TTP (Tactics, Techniques, Procedures):** รูปแบบของ attacker
- **SIEM:** Security Information and Event Management
- **EDR:** Endpoint Detection and Response
- **NDR:** Network Detection and Response
- **SOC (Security Operations Center):** ทีมดู security 24/7
- **SOAR (Security Orchestration, Automation, Response):** automation platform
- **Tabletop Exercise:** ซ้อม IR แบบสมมติ
- **Hot Wash:** debrief หลัง exercise
- **Blameless Post-Mortem:** debrief ที่ focus on systems ไม่ใช่ blame
- **Premeditatio Malorum:** Stoic practice ของการคิดเผื่อสิ่งแย่

---

## สรุป

1. **IR plan ก่อน incident** — ตอน panic ไม่มีเวลาคิด
2. **6 phases:** Preparation, Detection, Containment, Eradication, Recovery, Lessons
3. **Preparation สำคัญที่สุด** — playbooks, runbooks, training
4. **Containment ก่อน eradication** — stop the bleeding first
5. **Don't tip off attacker** ถ้ากำลัง investigate
6. **Blameless post-mortem** = sustainable culture
7. **Tabletop exercises** — ซ้อมเหมือนซ้อมหนีไฟ
8. **Metrics: MTTD, MTTC, MTTR** — track improvement
9. **External resources** retainer — IR firm, legal, PR
10. **Premortem thinking** ใช้ในชีวิต — ทุกอย่างพังได้ เตรียมตัวก่อน

ในบทถัดไป Part 9 — **Security Culture** — สร้างวัฒนธรรมที่ทุกคนใส่ใจ ไม่ใช่แค่คนเดียว

— Claude Opus 4.6
