# บทความที่ 22: Security Culture — ทำยังไงให้ทั้งทีมใส่ใจ ไม่ใช่แค่คนเดียว

**CyberSecurity Awareness Series — Part 9: Building Security Culture**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของบริษัทที่ "Security เก่งสุด แต่โดน Hack"

ในวงการ security มีคำพูดติดปาก:

> *"You can buy the best firewall, the best antivirus, the best SIEM — but if your culture doesn't care, you'll still get breached."*

ตัวอย่างชัดเจนที่สุดคือ **Yahoo!** — ในช่วง 2013-2016 บริษัทถูก hack:

- **3 พันล้านบัญชี** หลุด (ใช่ครับ — 3 billion, ไม่ใช่ million)
- ใหญ่ที่สุดในประวัติศาสตร์
- กระทบทุกบัญชี Yahoo ที่เคยมีอยู่

แต่ที่น่าตกใจไม่ใช่ขนาดของ breach — แต่คือ **Yahoo รู้ตั้งแต่ 2014 แต่ไม่บอกใคร**

ทำไม? เพราะ:

- **Marissa Mayer (CEO)** ลด security budget เพื่อ focus growth
- **CISO Alex Stamos** ขอ encryption end-to-end สำหรับ email — **ปฏิเสธ**
- **Internal security warnings** ถูก ignore เพื่อไม่ให้กระทบ user experience
- **Security team** ถูกเรียกว่า "**Paranoids**" ในเชิงดูถูก
- Stamos ลาออกในปี 2015 ไป Facebook (ที่ valued security culture มากกว่า)

ในปี 2016 เมื่อ Yahoo จะถูกซื้อโดย Verizon — Verizon **ลดราคาซื้อ $350 million** เพราะ breach ที่เพิ่งเปิดเผย

[อ่านรายละเอียด Yahoo breaches](https://en.wikipedia.org/wiki/Yahoo!_data_breaches)

> **Yahoo มี security technology ที่ดี — แต่ security culture ที่แย่ — ผลคือ disaster ที่ใหญ่ที่สุดในประวัติศาสตร์**

ตรงข้าม — บริษัทที่มี security culture ดี (เช่น Cloudflare, Stripe, GitHub) มักรอดจาก incident ที่ใหญ่ๆ — เพราะ **ทุกคนช่วยกันเฝ้าระวัง** ไม่ใช่แค่ security team

ในบทความนี้เราจะดู:

1. **ทำไม security ต้องเป็นเรื่องของทุกคน**
2. **Security Champions Program**
3. **Blameless Culture**
4. **Definition of Done** with security
5. **Activities** ที่ build culture
6. **Sustaining Culture** ในระยะยาว

---

## ทำไม Security ต้องเป็นเรื่องของทุกคน

### Security Team ไม่สามารถทำคนเดียวได้

ในบริษัทขนาด 500 คน — security team อาจมี 5-10 คน

- **5 security engineers** vs **490 dev/ops/sales/HR**
- ทุกคน **490** สามารถสร้าง vulnerability ได้
- Security team **5** ไม่มีทาง review ทุกอย่าง

ผลคือ — security team กลายเป็น **bottleneck** + **blocker** + **น่ารำคาญ** สำหรับคนอื่น

### "Shifting Left" Security

แนวคิดที่กล่าวในบทที่ 20 — **integrate security ตั้งแต่ต้น** แทนที่จะเป็นด่านสุดท้าย

แต่นี่ทำได้แค่ถ้า **dev เป็น part of security**:

```
ก่อน: Security team review → Block → Conflict
หลัง: Dev think security → Self-review → Security team partner
```

### 99% ของ Breaches เกิดจาก "Human Element"

**Verizon DBIR 2024** — 68% of breaches involve human element:
- Phishing
- Misconfiguration
- Stolen credentials  
- Privilege misuse

→ Technology อย่างเดียวแก้ไม่ได้ — ต้อง **change behavior**

### Security Culture vs Security Compliance

#### Compliance Culture (ผิวเผิน)

```
"We follow policy because we have to."
"Security checklist? Done. Now let's ship."
"That's security team's problem."
```

#### Security Culture (ลึก)

```
"We follow practices because we understand the risk."
"How does this affect security? Let's discuss."
"Security is everyone's job."
```

**ความต่าง** — มาจาก "fear-based" หรือ "value-based"?

---

## Security Champions Program

นี่คือกลยุทธ์ที่ effective ที่สุด — สร้าง **security ambassadors** ในทุก team

### What is a Security Champion

**Security Champion** = developer (หรือ engineer ใน team อื่น) ที่:

- Interest ใน security
- ไม่ใช่ security professional แบบ full-time
- ทำงานปกติของตัวเอง 80% + advocate security 20%
- **Bridge** ระหว่าง security team และ dev team

### Why Champions Work

1. **Embedded ใน team** — เข้าใจ context ของ team
2. **Speak the language** — ของ dev, ไม่ใช่ "security speak"
3. **Trusted** — เป็นเพื่อนร่วมทีม, ไม่ใช่ outsider
4. **Scale** — security team 10 คน → champions 50 คน → coverage 5x

### Champion Responsibilities

- **Review code** ของ team จากมุม security
- **Threat model** ของ feature ใหม่
- **Triage security alerts** ที่กระทบ team
- **Train teammates** เรื่อง security basics
- **Liaison** กับ security team
- **Stay updated** เรื่อง vulnerability + best practices

### Setting Up the Program

#### Step 1: Recruit (Don't Force)

✅ **Volunteer-based** — คนที่อยากเรียน security จริงๆ  
❌ **Assigned** — คนที่ถูกบังคับ → resentment

```
Email to dev team:
---
Hi team,

Want to grow your skills in security? Join our Security Champions program!

What you get:
- Dedicated training (40 hours/year)
- Conference budget ($2K/year)
- Mentorship from security team
- Recognition + bonus
- Career growth path

What you do:
- Be your team's security advocate (4-8 hours/week)
- Review code from security perspective
- Help triage security findings
- Bring security topics to team meetings

Interest? Reply by Friday.
```

#### Step 2: Train

ก่อนรับ responsibility — provide training:

- **Onboarding** week (security basics)
- **OWASP Top 10** workshop (chapter 14)
- **Threat modeling** workshop (chapter 20)
- **Incident response** training (chapter 21)
- **Tool proficiency** — SIEM, EDR, etc.

#### Step 3: Empower

- **Dedicated time** — 20% allocation
- **Authority** — can request changes
- **Resources** — tools, training budget
- **Recognition** — visibility, promotion path

#### Step 4: Connect

```
Weekly: Champions meeting (1 hour)
  - Share challenges
  - Cross-team learnings
  - Q&A with security team

Monthly: All-Champions meeting (2 hours)
  - New threats
  - Tool updates
  - Best practices

Quarterly: Champions Day
  - Hackathon
  - CTF
  - Presentations
```

#### Step 5: Measure

KPIs ของ program:

- Vulnerability detection rate per team (with vs without champion)
- Time to resolve security findings
- Code coverage by security review
- Champion retention rate
- Champion → security engineer conversion rate

### Champion Career Path

```
Year 1: Security Champion (part-time)
   ↓
Year 2: Senior Champion + lead initiatives
   ↓
Year 3: Move to Security team OR Security-focused PM/EM role
   ↓
Year 4+: Security architect / specialist
```

→ Champions เป็น pipeline สำหรับ security team

### Common Mistakes

#### 1. ไม่มี Time Allocation

> "ใช่, เป็น champion ก็ดี แต่ทำงานปกติของตัวเองด้วยนะ"

→ Champion หมดไฟ → quit

✅ **Solution:** Dedicate 20% time ที่ manager ยอมรับ

#### 2. ไม่มี Authority

> "Champion แค่ "advise" — manager ตัดสินใจ"

→ Champion ทำได้แค่บ่น

✅ **Solution:** Champion มี authority เรียก security review, block deploy ถ้าจำเป็น

#### 3. Champion ที่ Wrong People

> "ทุกคนต้องเป็น champion"

→ Forced — ไม่ effective

✅ **Solution:** Volunteer + interview screening

---

## Blameless Culture

ที่กล่าวในบทที่ 21 — แต่ลึกกว่านั้น

### Why Blame Doesn't Work

#### Blame Creates Hiding

```
Engineer: "I think I might have introduced a security bug"
   ↓
Blame culture: "If I report this, I get fired/yelled at"
   ↓
Engineer: stays silent
   ↓
Bug stays in production until exploited
```

#### Blame Creates Fear

Fear-based culture:
- People don't try new things (innovation dies)
- People don't admit mistakes (repeats)
- People hide problems (blow up later)
- Best people leave (only fearful stay)

### Just Culture Framework

จาก aviation industry — categorize behavior, not outcome:

#### Human Error (innocent mistake)

- "I clicked the wrong button"
- "I typed the IP wrong"
- "I forgot to verify"

→ **Console + improve system** — don't blame

#### At-Risk Behavior (taking shortcut, didn't realize risk)

- "I skipped the review because I was rushing"
- "I copied the password into chat"
- "I disabled MFA because it was annoying"

→ **Coach + improve training**

#### Reckless Behavior (knowingly violating)

- "I knew the policy but did it anyway"
- "I bypassed the security check intentionally"
- "I shared credentials despite being told not to"

→ **Disciplinary action** — but rare!

→ **>95% of incidents = human error or at-risk** — system fix, not blame

### Blameless Reporting Channels

#### "See Something, Say Something"

```
Slack channel: #security-concerns
  - Anonymous reporting OK
  - "I clicked a phishing link" → no shame
  - "I think I committed a secret" → fast response
  - "Something looks weird" → investigated

Reward: thank you (publicly if person OK)
Punishment: never for honest reports
```

#### Bug Bounty (Internal)

Reward employees for finding security issues:

```
Severity     Reward
Critical     ฿20,000
High         ฿10,000
Medium       ฿3,000
Low          ฿500
```

→ Encourage finding + reporting

#### "Phishing Reporter Hero"

Recognize people who report phishing:

```
Slack: "🏆 Security Hero of the Week: Tanin reported a phishing 
       attempt that targeted our finance team. Quick reporting 
       prevented potential breach!"
```

### Blameless Post-Mortems (Refresher)

จากบทที่ 21:

```
Bad: "John failed to apply the patch on time."

Good: "The patch monitoring alert went to a Slack channel 
that John wasn't subscribed to. We'll fix:
1. Auto-subscribe on-call to monitoring channels
2. Add patch SLA dashboard
3. Pair new on-call with experienced for first 2 weeks"
```

---

## Security as Part of Definition of Done

> **"It's not done until it's secure."**

### Definition of Done — Traditional

```
- [ ] Code written
- [ ] Tests pass
- [ ] Reviewed by 1 person
- [ ] Documentation updated
- [ ] Deployed to staging
```

### Definition of Done — Security-Aware

```
- [ ] Code written
- [ ] Tests pass (incl. security tests)
- [ ] Reviewed by 1 person + security champion
- [ ] Documentation updated (incl. threat model)
- [ ] Static analysis passed (Semgrep, etc.)
- [ ] Dependency scan passed (npm audit, Snyk)
- [ ] Secret scan passed (gitleaks)
- [ ] Inputs validated
- [ ] Authn/authz checked
- [ ] PII handled per privacy policy
- [ ] Logging without sensitive data
- [ ] Deployed to staging
- [ ] Penetration tested (if SEV-1)
```

### How to Make It Stick

#### 1. PR Template

```markdown
## Description
[What does this PR do?]

## Security Checklist
- [ ] Inputs validated
- [ ] No new secrets in code
- [ ] Existing tests still pass
- [ ] New auth/authz logic reviewed
- [ ] Threat model updated (if needed)
- [ ] Logs don't contain PII

## Threat Model Considerations
[STRIDE quick analysis: any new spoofing/tampering/etc. risks?]
```

#### 2. CI/CD Integration

```yaml
# Block PR if any security check fails
- gitleaks (secret scan)
- npm audit / pip-audit (vulnerable deps)
- Semgrep (static analysis)
- Dependency review action
```

#### 3. Required Reviewers

```yaml
# CODEOWNERS
# Security-sensitive paths require security review
/auth/         @security-team
/payments/     @security-team
/admin/        @security-team
```

#### 4. Sprint Planning

```
- Story 1: Add user profile feature
   - Sub-task: Threat model
   - Sub-task: Security review
   - Sub-task: Update privacy notice if needed
```

---

## Security Awareness Activities

ก่อสร้าง culture ผ่าน activities ที่ทุกคน participate

### 1. Capture The Flag (CTF)

#### Internal CTF

ทำเป็น company event ที่ปีละ 1-2 ครั้ง:

```
Format:
  - Team-based
  - 1 day event
  - Mix of: web, network, crypto, forensics, OSINT
  - Beginner-friendly + advanced challenges
  - Prizes for top teams + "best newbie"

Benefits:
  - Hands-on learning
  - Build security mindset
  - Team bonding
  - Recognition
```

#### External CTFs

Sponsor team to compete:

- **DEF CON CTF** — most prestigious
- **Google CTF**
- **Pwn2Own** — bug bounty event
- **HITB CTF** — Hack In The Box
- Local Thai CTFs

### 2. Phishing Simulation

ส่ง fake phishing emails ทดสอบ team:

#### Tools

- **KnowBe4** — most popular commercial
- **Hoxhunt** — modern UX
- **Proofpoint** — enterprise
- **PhishMe** (Cofense)
- **GoPhish** — open source

#### Process

```
Monthly:
  - Send phishing campaign to 10-20% of employees
  - Click rates tracked (without naming individuals)

Click → 
  - Land on training page (not blame)
  - Quick lesson on what they missed
  - Tip for next time

Report → 
  - Praise (publicly if person OK)
  - Statistics shared

Quarterly:
  - Report click rate trend
  - Adjust training based on which phish work
```

#### Goal

- Year 1: Phish click rate < 30%
- Year 2: < 15%
- Year 3: < 5%
- **Reporting rate** > 70% (more important!)

### 3. Lunch & Learn

```
Monthly 30-min session:
  - Topic: "What is a Zero Day"
  - Topic: "Anatomy of a Phishing Email"
  - Topic: "How LastPass got hacked"
  - Topic: "AI in security 2026"

Format:
  - Lunch provided (or stipend)
  - Casual atmosphere
  - Q&A encouraged
  - Recording for absent
```

### 4. Security Newsletter

Weekly/monthly:

```markdown
# Security Weekly — Week 17, 2026

## In the News
- Microsoft patches critical Outlook RCE
- ...

## Internal Updates
- New phishing simulation results: 12% click rate (down from 18%)
- Security champion of the week: Lisa from Backend team

## Tip of the Week
"Always verify email sender by hovering over the address — typosquatting is real."

## Bug Bounty Leaderboard
1. Tanin — 5 valid reports
2. Mai — 3 valid reports

## Upcoming
- Lunch & Learn next Friday: "AI Prompt Injection"
- CTF event in 3 weeks
```

### 5. New Hire Security Training

Day 1-2 of onboarding:

```
- Company security policy
- Common threats (phishing, social engineering)
- Password manager setup
- 2FA setup
- VPN / ZTNA setup
- "If you see something, say something"
- Hands-on: spot phishing email exercise
- Sign acceptable use policy
```

### 6. Annual Compliance Training

Required for everyone:

- 1-hour online module
- Refresher on policies
- New threats covered
- Quiz at end
- Track completion in HR system

### 7. Security Slack Channel

Always-on channel `#security-tips`:

- Daily tip posts
- Q&A welcome
- Memes encouraged (make security fun)
- News sharing

### 8. "Security Office Hours"

Weekly slot when security team available:

```
Every Tuesday 14:00-15:00:
  - Drop by Slack channel
  - Ask anything
  - No question is dumb
  - Discord-style voice if needed
```

### 9. Bug Bounty (External)

- **HackerOne** / **Bugcrowd** — manage program
- Pay researchers for finding vulnerabilities
- Public scope of what's allowed
- Disclosure policy

→ Crowdsource security testing

### 10. Security All-Hands

Quarterly:

- Security state of the company
- Major incidents (anonymized)
- Lessons learned
- Recognition of contributions
- New initiatives

---

## Sustaining Culture Long-Term

### Make It Easy

> **The easy way must be the secure way.**

ตัวอย่าง:

❌ **Hard:** "Use password manager — but it requires admin install + IT ticket"  
✅ **Easy:** Pre-installed password manager via MDM, just login

❌ **Hard:** "Use MFA — but you have to set it up yourself"  
✅ **Easy:** MFA enrollment as part of onboarding flow

❌ **Hard:** "Encrypt your data — figure out how"  
✅ **Easy:** Encryption by default in tools team uses

### Reward + Recognize

Public recognition matters:

```
- Slack shoutouts
- Security Champion of the Quarter award
- Bonus tied to security KPIs
- Promotion criteria include security
- Conference budget for security work
```

### Don't Compromise When Inconvenient

Tests of culture come during pressure:

#### Bad

> "We have a deadline. Skip the security review this once."

→ One time becomes habit

#### Good

> "We have a deadline. Let's do an expedited security review (1 day) instead of full review (3 days). What's the minimum we can ship safely?"

→ Maintains principle while pragmatic

### Lead by Example

Leadership behavior > policy:

- Does CEO use 2FA? Public knowledge.
- Does CTO follow code review? Visible.
- Does CISO admit mistakes? Models culture.

> **Culture flows from the top — actions speak louder than words**

### Iterate the Culture

Not a one-time setup:

```
Quarterly:
  - Survey team on security culture
  - Review metrics
  - Identify gaps
  - Adjust programs

Annually:
  - Strategic review
  - Major changes
  - Reset goals
```

### Cross-pollinate with Industry

- Send people to **DEF CON, Black Hat, RSA**
- Host **meetups** at office
- **Open source** security tools (build credibility)
- **Speak at conferences**

---

## Anti-Patterns to Avoid

### 1. Security Theater

> Doing things that look like security but don't actually help

Examples:
- 100-page password policy ที่ไม่มีใครอ่าน
- Annual training ที่ click-through ไม่ใส่ใจ
- Audit checkboxes ที่ไม่มี follow-through

→ Wastes time, builds cynicism

### 2. Security as Roadblock

> Security team always says "no"

```
Dev: "Can we use this new tool?"
Security: "No."
Dev: "Why?"
Security: "Because policy."
Dev: [shadow IT begins]
```

→ **Solution:** "Yes, with conditions" — partner instead of block

### 3. Fear-Mongering

> Security communications focus on scaring people

```
Email: "WARNING: A breach could DESTROY this company! 
Be VIGILANT or face CONSEQUENCES!"
```

→ Fear fatigue, people tune out

✅ **Better:** "Here's what we learned from XYZ Inc's breach last week, and three things you can do differently."

### 4. "Security Knows Best"

> Security team operates in isolation

Result: solutions ไม่ work for actual users

✅ **Better:** Co-design with users

### 5. Tools Over People

> Buying $$$ tools without changing behavior

```
"We bought CrowdStrike. We're now secure!"
```

→ Tools don't change human behavior

### 6. Annual Training Only

> Once a year, click through training, forget

→ No retention

✅ **Better:** Continuous reinforcement (newsletters, lunch & learn, simulations)

---

## Measuring Culture

### Culture Metrics

ยากกว่า technical metrics — แต่ทำได้:

#### 1. Phishing Click Rate (trending down?)

#### 2. Phishing Reporting Rate (trending up?)

#### 3. Security Concerns Raised

```
"How many security concerns reported this quarter?"
- Trending up = culture improving (more reports)
- Trending down = could be stagnating or hiding
```

#### 4. Time to Patch

```
- SEV-1: target 24h, actual ?
- SEV-2: target 7d, actual ?
```

#### 5. Survey Results

```
1-5 scale, anonymous:
- "I feel comfortable reporting security issues"
- "Security team is helpful, not blocker"
- "I understand security responsibilities of my role"
- "Security is taken seriously here"
```

#### 6. Security Champion Retention

If champions leave → culture issue

#### 7. Security in Hiring

How many candidates ask about security culture? (good sign of reputation)

---

## Action Items

### วันนี้

- [ ] **Self-assess:** มี security culture ที่บริษัทคุณไหม? Compliance หรือ value-based?
- [ ] **Talk to manager:** อยากเป็น security champion ไหม?
- [ ] **Subscribe** ข่าว security ที่ relevant กับ stack ของคุณ

### สัปดาห์นี้

- [ ] **Identify gaps** — checklist ใน DoD ของทีม
- [ ] **Setup #security-concerns** Slack channel
- [ ] **Schedule** lunch & learn topic แรก

### เดือนนี้

- [ ] **Run first phishing simulation** (or analyze recent)
- [ ] **Recruit champions** — 1 ต่อทุก 10-20 dev
- [ ] **Update PR template** with security checklist

### สำหรับ Tech Lead / EM

- [ ] **Champion** for your team — recruit + support
- [ ] **DoD update** with security items
- [ ] **Time allocation** — 20% for champion
- [ ] **Recognize** security work in 1:1s

### สำหรับ CISO / Security Lead

- [ ] **Champion program** structure
- [ ] **Quarterly culture survey**
- [ ] **Internal CTF** schedule
- [ ] **Bug bounty** (internal first, then external)
- [ ] **Newsletter** weekly cadence
- [ ] **Office hours** weekly

### สำหรับ CEO / Executive

- [ ] **Public commitment** to security
- [ ] **Lead by example** — use 2FA, MFA, follow policies
- [ ] **Resource allocation** for security
- [ ] **No compromise** when inconvenient
- [ ] **Reward** security work in compensation

---

## บทเรียนชีวิตจากบทความนี้

> **วัฒนธรรมที่ดีสร้างจากการทำให้เป็นเรื่องปกติ ไม่ใช่เรื่องน่ากลัว**

หลักการสร้าง culture — ใช้ได้ทุกที่ในชีวิต:

#### ครอบครัว

**Bad culture:**
- ลูกพลาด → ตี
- ลูกถาม → "อย่าถามมาก"
- ลูกขอความช่วยเหลือ → "เรื่องเล็กแก้เอง"

→ ลูกซ่อนปัญหา → ปัญหาโตขึ้น

**Good culture:**
- ลูกพลาด → "เกิดอะไรขึ้น?" (เข้าใจก่อน)
- ลูกถาม → "ดีที่ถาม" (encourage)
- ลูกขอความช่วยเหลือ → "พร้อมช่วยเสมอ"

→ ลูกบอกปัญหาเร็ว → แก้ได้

#### ทีมงาน

**Bad culture:**
- พลาด → ถูกตำหนิ
- เสนอไอเดีย → ถูกล้อ
- ถามคำถาม "พื้นฐาน" → ถูกบอก "ควรรู้แล้วสิ"

→ คนเก็บปัญหาไว้คนเดียว, ไม่ขอความช่วยเหลือ

**Good culture:**
- พลาด → "เรียนรู้อะไรจากนี้บ้าง?"
- เสนอไอเดีย → "Let's explore"
- คำถาม → "Good question"

→ Innovation เกิด, ปัญหาแก้เร็ว

#### Health Habits

**Bad culture (Self):**
- "ออกกำลังกาย เป็นเรื่องน่ากลัว เหนื่อย เจ็บ"
- "กินดี ขี้เกียจ"
- "Sleep ดี เสียเวลา"

→ ใจไม่อยากทำ → ไม่ทำ → สุขภาพแย่

**Good culture:**
- "ออกกำลังกาย เป็นกิจวัตรเหมือนแปรงฟัน"
- "กินดี เป็นการดูแลตัวเอง"
- "Sleep ดี เป็น investment"

→ ทำเป็นปกติ → ไม่ต้องคิด → สุขภาพดี

> **วัฒนธรรม = "การทำเรื่องดีให้กลายเป็นเรื่องปกติ"**

ในงาน security:

- **Bad:** "MFA น่ารำคาญ ต้องบังคับ"
- **Good:** "MFA เป็น standard เหมือน lock door"

- **Bad:** "Code review เสียเวลา"  
- **Good:** "Code review เป็นโอกาสเรียน"

- **Bad:** "Threat modeling เป็นภาระ"
- **Good:** "Threat modeling ทำให้เรา confident ว่า code ปลอดภัย"

### Atomic Habits Approach

จากหนังสือ **"Atomic Habits"** ของ James Clear — การเปลี่ยน habit ผ่าน 4 laws:

1. **Make it Obvious** — visible, easy to start
2. **Make it Attractive** — fun, social, rewarding
3. **Make it Easy** — minimum friction
4. **Make it Satisfying** — immediate reward

ในเรื่อง security culture:

| Law | Application |
|---|---|
| **Obvious** | Security checklist ใน PR template, dashboards visible |
| **Attractive** | CTF events, recognition, prizes |
| **Easy** | Pre-configured tools, automated checks |
| **Satisfying** | Bonus for findings, public praise |

### "Default" Culture

> **Culture is determined by what's "default" — not what's "exceptional"**

- ถ้า default = follow security → ทำตามเป็นปกติ
- ถ้า default = skip security → security เป็นเรื่อง exception

→ Tweak defaults!

### Long Game

Culture **ไม่เปลี่ยนได้ในเดือนเดียว** — ใช้เวลาเป็นปี

```
Year 1: Foundation (basic awareness, tools)
Year 2: Engagement (champions, activities, simulations)
Year 3: Embedded (security in everyday work)
Year 4+: Mature (continuous improvement)
```

> **คนที่เริ่มต้น สร้าง culture ดี — ทิ้งมรดก ที่ใช้กันต่อไปเป็นปี**

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไป — **บทสุดท้าย** — สรุปทุกอย่าง + checklist ที่ทุกคนทำได้ทันที

---

## อภิธานศัพท์ (Glossary)

- **Security Culture:** values + behaviors ของ team เกี่ยวกับ security
- **Security Champion:** dev ที่ advocate security ในทีมตัวเอง
- **Just Culture:** framework ที่ categorize behavior (error/at-risk/reckless)
- **Blameless Post-Mortem:** debrief that focuses on systems, not people
- **Definition of Done (DoD):** criteria ว่า task เสร็จเมื่อไหร่
- **CTF (Capture The Flag):** security competition
- **Phishing Simulation:** fake phishing emails for training
- **Bug Bounty:** program ที่จ่ายเงินสำหรับ vulnerability
- **Security Theater:** activities ที่ดูเหมือน security แต่ไม่ effective
- **Shadow IT:** unauthorized tools ที่ employees ใช้
- **DBIR (Data Breach Investigations Report):** Verizon's annual report
- **Atomic Habits:** book by James Clear about habit formation
- **Security Office Hours:** scheduled time security team available

---

## สรุป

1. **Yahoo case** — เทคโนโลยีดี + culture แย่ = disaster
2. **Security ต้องเป็นเรื่องของทุกคน** — security team อย่างเดียวไม่พอ
3. **Security Champions** — embed advocates ในทุก team
4. **Blameless culture** — encourage reporting ไม่ใช่ hiding
5. **Definition of Done** with security — ฝัง security ใน normal workflow
6. **Activities** = CTF, phishing simulation, lunch & learn, newsletter
7. **Make it easy** — easy way = secure way
8. **Lead by example** — leadership behavior > policy
9. **Measure culture** — phishing rates, surveys, retention
10. **Long game** — culture takes years to build, but lasts longer

ในบทถัดไป — **บทสุดท้าย** — Personal Security Checklist ที่ทุกคนทำได้ทันที

— Claude Opus 4.6
