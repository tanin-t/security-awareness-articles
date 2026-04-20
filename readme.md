# CyberSecurity Awareness Series — Outline

> **กลุ่มเป้าหมาย:** Dev Team  
> **แนวคิดหลัก:** Cybersecurity ไม่ใช่แค่ทักษะคอมพิวเตอร์ แต่เป็น **life skill** — เป็นเรื่องของการบริหารความเสี่ยง ความไม่ประมาท และการคิดอย่างเป็นระบบ ที่ใช้ได้ทั้งในงานและชีวิตประจำวัน  
> **โทน:** Semi-formal — เข้าถึงง่าย แต่จริงจังพอสำหรับแชร์ทั้งองค์กร  
> **ภาษา:** ไทย ผสม English technical terms

---

## Part 1 — Foundation: เปลี่ยน Mindset ก่อนเปลี่ยน Password

### [บทความที่ 1: ทำไม Cybersecurity ถึงเป็น Life Skill ไม่ใช่แค่ IT Skill](articles/01-cybersecurity-is-a-life-skill.md)

- ทำไมคนทั่วไป (ไม่ใช่แค่ dev) ถึงต้องสนใจเรื่องนี้
- Cybersecurity คือ **risk management** — เหมือนการล็อกบ้าน ใส่หมวกกันน็อค หรือตรวจสุขภาพ
- แนวคิด "assume breach" — ไม่ใช่เรื่องของ *ถ้า* โดน แต่เป็น *เมื่อ* โดน
- ตัวอย่างเหตุการณ์จริงที่เกิดจากความประมาทเล็กๆ น้อยๆ
- **Life lesson:** ความประมาทในเรื่องเล็กๆ สะสมเป็นหายนะได้ — ใช้ได้กับทุกเรื่องในชีวิต

---

## Part 2 — Password & Authentication: ด่านแรกที่คนส่วนใหญ่แพ้

### บทความที่ 2: ทำไมห้ามใช้ Password ซ้ำ — และผลลัพธ์ที่คุณไม่อยากเจอ

- **Credential stuffing** คืออะไร — เมื่อ password ที่หลุดจากเว็บหนึ่ง ถูกใช้เจาะอีกร้อยเว็บ
- ตัวอย่างจาก data breach ที่เกิดขึ้นจริง (เช่น RockYou2024, Collection #1)
- Demo: แสดงให้เห็นว่า attacker ใช้ leaked credentials อย่างไร
- วิธีเช็คว่า email/password ของคุณหลุดไปแล้วหรือยัง (haveibeenpwned.com)
- **Life lesson:** อย่าวาง "กุญแจดอกเดียว" ไว้กับทุกอย่าง — กระจายความเสี่ยง

### บทความที่ 3: หยุดส่งและเก็บ Plaintext Password ได้แล้ว

- ทำไม plaintext password ถึงอันตราย — ทั้งในฐานะ user และ developer
- กรณีศึกษา: บริษัทที่เก็บ password เป็น plaintext แล้วหลุด
- สำหรับ dev: **hashing** (bcrypt, Argon2) vs encryption vs plaintext — อะไรควรใช้เมื่อไหร่
- สำหรับ user: อย่าส่ง password ผ่าน chat, email, LINE, shared doc
- วิธีส่ง credentials อย่างปลอดภัย (one-time secret links, password manager sharing)
- **Life lesson:** ข้อมูลสำคัญต้องมี "ซองปิดผนึก" เสมอ — ไม่ว่าจะเป็นดิจิทัลหรือชีวิตจริง

### บทความที่ 4: Password Manager — ทำไมต้องใช้ และเริ่มใช้ยังไง

- ปัญหาของการจำ password เอง — สมองมนุษย์ไม่ได้ออกแบบมาเพื่อเรื่องนี้
- Password manager ทำงานยังไง (master password, vault, encryption)
- เปรียบเทียบตัวเลือก: 1Password, Bitwarden, KeePass — เลือกอะไรดี
- **Hands-on guide:** ขั้นตอนการ setup + migrate passwords ที่มีอยู่
- Team sharing: วิธีแชร์ credentials ในทีมอย่างปลอดภัย
- **Life lesson:** ใช้ระบบช่วยจัดการสิ่งที่ซับซ้อน แทนที่จะพึ่งพาความจำ

### บทความที่ 5: 2FA / MFA — เลเยอร์ที่สองที่ช่วยชีวิตคุณ

- 2FA คืออะไร — something you know + something you have
- ประเภทของ 2FA: SMS OTP vs Authenticator App vs Hardware Key (YubiKey)
- ทำไม SMS OTP ถึงไม่ปลอดภัยเท่าที่คิด (SIM swapping, SS7 attack)
- แนะนำ: ใช้ TOTP app (Google Authenticator, Authy) หรือ passkey
- **Recovery codes** — สำคัญเท่า 2FA เอง อย่าลืมเก็บ
- **Hands-on:** เปิด 2FA ให้ GitHub, Google, Slack ทันที
- **Life lesson:** ระบบป้องกันที่ดีมีหลายชั้น — เหมือนบ้านที่มีทั้งรั้ว กลอน และกล้อง

---

## Part 3 — Cryptography พื้นฐาน: ภาษาลับของโลกดิจิทัล

### บทความที่ 6: Cryptography 101 — เข้าใจ Encryption, Hashing, และ Digital Signatures

- ทำไม dev ต้องเข้าใจ cryptography พื้นฐาน (ไม่ต้องเป็นนักคณิตศาสตร์)
- **Symmetric vs Asymmetric encryption** — อธิบายด้วยภาษาคน
  - Symmetric: AES — กุญแจดอกเดียวทั้งล็อกและปลดล็อก
  - Asymmetric: RSA, ECC — public key / private key คืออะไร
- **Hashing** vs **Encryption** — ต่างกันยังไง ใช้เมื่อไหร่
  - Hashing: SHA-256, bcrypt — ทางเดียว ย้อนกลับไม่ได้
  - Encryption: ย้อนกลับได้ถ้ามี key
- **Digital signatures** — พิสูจน์ว่าข้อมูลมาจากใคร และไม่ถูกแก้ไข
- **TLS/SSL** — เกิดอะไรขึ้นจริงๆ เมื่อเปิดเว็บ HTTPS
  - Certificate chain, Certificate Authority (CA)
  - ทำไม self-signed cert ถึงเตือน
- **Encryption at rest vs in transit** — ปกป้องข้อมูลทั้งตอนเก็บและตอนส่ง
- Key management พื้นฐาน — key rotation, key storage, อย่า hardcode key
- Common mistakes: ใช้ MD5/SHA1, implement crypto เอง, ลืม salt
- **Life lesson:** คุณไม่จำเป็นต้องรู้ว่าเครื่องยนต์ทำงานยังไงทุกชิ้นส่วน แต่ต้องรู้ว่าเมื่อไหร่ควรเติมน้ำมัน เมื่อไหร่ควรเปลี่ยนยาง — cryptography ก็เช่นกัน

---

## Part 4 — Everyday Threats: ภัยที่เจอได้ทุกวัน

### บทความที่ 7: Phishing & Social Engineering — เมื่อคนคือช่องโหว่ที่ใหญ่ที่สุด

- Phishing คืออะไร — ไม่ใช่แค่อีเมลหลอก แต่มาในทุกรูปแบบ (email, SMS, voice call, QR code)
- **Spear phishing** — การโจมตีแบบเจาะจงเป้าหมาย
- Social engineering tactics: urgency, authority, fear, curiosity
- วิธีสังเกต phishing: URL inspection, sender verification, too-good-to-be-true
- ถ้าคลิกไปแล้ว ทำอะไรต่อ? (incident response สำหรับตัวเอง)
- **Life lesson:** ระวังคนที่พยายามกดดันให้คุณตัดสินใจเร็วๆ — ไม่ว่าจะเป็นอีเมล มิจฉาชีพ หรือ sales pressure

### บทความที่ 8: Secure Your Devices — Laptop, Phone, และ อุปกรณ์ทุกชิ้นที่คุณใช้

- Disk encryption (FileVault, BitLocker) — ทำไมต้องเปิด
- Screen lock & auto-lock timeout
- Software updates — ทำไมต้อง update ทันที (patch Tuesday, zero-day)
- Public WiFi risks — และทำไม VPN ถึงจำเป็น
- Physical security: อย่าทิ้ง laptop ไว้ใน coffee shop
- Mobile device management สำหรับทีม
- **Life lesson:** ดูแลเครื่องมือของคุณ — เหมือนดูแลรถยนต์ ต้อง maintain ไม่ใช่แค่ใช้

### บทความที่ 9: Safe Browsing & Online Hygiene — พฤติกรรมออนไลน์ที่ปลอดภัย

- Browser extensions ที่ควรมีและที่ควรลบ
- HTTPS everywhere — ดู padlock ไม่พอ ต้องรู้จัก certificate
- Cookie consent ≠ cookie safety — third-party tracking
- การจัดการ browser permissions (location, camera, mic)
- Disposable email, alias email สำหรับ sign up บริการที่ไม่สำคัญ
- **Life lesson:** ทุกครั้งที่กด "Allow" หรือ "Accept" ให้ถามตัวเองว่ากำลังแลกอะไรอยู่

### บทความที่ 10: Ransomware — เมื่อข้อมูลของคุณถูกจับเป็นตัวประกัน

- Ransomware คืออะไร — กลไกการทำงาน (encryption → ransom note → payment)
- วิวัฒนาการ: จาก WannaCry สู่ **Ransomware-as-a-Service (RaaS)** และ **double extortion**
- กรณีศึกษาที่สำคัญ:
  - WannaCry (2017) — ล้มระบบ NHS, โรงงาน, ทั่วโลก
  - Colonial Pipeline (2021) — ท่อส่งน้ำมันหยุดทำงาน
  - กรณีในไทย: โรงพยาบาล, หน่วยงานรัฐ
- **Attack vectors** ที่พบบ่อย: phishing email, RDP exposed, unpatched vulnerabilities
- จ่ายค่าไถ่ดีไหม? — ทำไมผู้เชี่ยวชาญส่วนใหญ่บอกว่าไม่ควร
- วิธีป้องกัน:
  - Patch ทันที, segment network
  - จำกัด admin privileges
  - Email filtering & endpoint protection
  - **Backup** คือเกราะสุดท้าย (→ อ่านต่อบทความถัดไป)
- ถ้าโดน ransomware แล้ว — ขั้นตอนแรกที่ต้องทำทันที
- **Life lesson:** บางครั้งสิ่งที่ช่วยคุณไม่ใช่กำแพงที่สูงที่สุด แต่เป็นแผนสำรองที่เตรียมไว้

### บทความที่ 11: Backup & Disaster Recovery — แผนสำรองที่ทุกคนต้องมี

- ทำไม backup ถึงเป็น "ประกันชีวิต" ของข้อมูล — ไม่ใช่แค่เรื่อง ransomware
- **กฎ 3-2-1 Backup Rule:**
  - 3 copies ของข้อมูล
  - 2 storage types ที่ต่างกัน
  - 1 copy อยู่ offsite / offline
- ประเภทของ backup: Full, Incremental, Differential — เลือกใช้ยังไง
- Backup สำหรับ developer:
  - Git ≠ backup — ทำไม repo อย่างเดียวไม่พอ
  - Database backup strategies (logical vs physical, point-in-time recovery)
  - Infrastructure as Code — rebuild ได้ทุกเมื่อ
- Backup สำหรับชีวิตส่วนตัว:
  - รูปภาพ, เอกสารสำคัญ, ข้อมูลมือถือ
  - Cloud backup vs local backup
- **Disaster Recovery Plan** — backup ไม่มีค่าถ้า restore ไม่ได้
  - RTO (Recovery Time Objective) & RPO (Recovery Point Objective)
  - ทดสอบ restore เป็นประจำ — "backup ที่ไม่เคยทดสอบ ไม่ใช่ backup"
- **Hands-on:** ตั้ง automated backup สำหรับ database + personal data วันนี้
- **Life lesson:** คนฉลาดเตรียมร่มก่อนฝนตก — backup คือร่มของโลกดิจิทัล สำเนาเอกสาร พินัยกรรม ประกัน ก็เป็น backup ของชีวิต

---

## Part 5 — Developer-Specific: สิ่งที่ Dev ต้องรู้เป็นพิเศษ

### บทความที่ 12: Secrets Management — อย่า Hardcode ความลับลงใน Code

- กรณีศึกษา: API keys, credentials หลุดผ่าน GitHub public repos
- `.env` files, `.gitignore` — พื้นฐานที่ต้องทำ
- Secret management tools: HashiCorp Vault, AWS Secrets Manager, doppler
- Pre-commit hooks สำหรับ detect secrets (gitleaks, trufflehog)
- Secret rotation — ทำไมต้องเปลี่ยน key เป็นประจำ
- **Life lesson:** อย่าเก็บของมีค่าไว้ในที่ที่คนทั่วไปเข้าถึงได้

### บทความที่ 13: Dependency & Supply Chain Attack — เมื่อสิ่งที่คุณ Trust กลับทำร้ายคุณ

- Supply chain attack คืออะไร — attack ผ่านสิ่งที่คุณเชื่อถือ (library, tool, vendor)
- กรณีศึกษา: SolarWinds, event-stream (npm), codecov, xz-utils backdoor
- **Typosquatting** — package ปลอมชื่อคล้ายของจริง
- วิธีป้องกัน:
  - ใช้ lock files (`package-lock.json`, `Pipfile.lock`)
  - ตรวจ dependency ด้วย `npm audit`, `pip-audit`, Dependabot, Snyk
  - Pin versions, verify checksums
  - Review ก่อน update — อย่า blindly upgrade
- Software Bill of Materials (SBOM) — ทำไมต้องรู้ว่าคุณใช้อะไรบ้าง
- **Life lesson:** Trust แต่ Verify — เชื่อใจได้ แต่ต้องตรวจสอบเสมอ ใช้ได้กับคน สินค้า และสัญญา

### บทความที่ 14: Secure Coding Practices — เขียนโค้ดแบบคนที่รู้ว่ามีคนจะ Hack

- OWASP Top 10 — ภาพรวมช่องโหว่ที่พบบ่อยที่สุด
- Input validation & sanitization — ไม่ trust input จาก user เด็ดขาด
- SQL Injection, XSS, CSRF — ยังอยู่และยังอันตราย
- Principle of least privilege ในการเขียนโค้ด
- Logging & monitoring — log อะไรบ้าง อะไรห้าม log (PII, secrets)
- Code review with security lens — checklist สำหรับ reviewer
- **Life lesson:** ออกแบบทุกอย่างโดยคิดว่ามีคนจะหาช่องว่าง — ใช้ได้กับ contract, กฎ, และระบบทุกชนิด

### บทความที่ 15: API Security — ปิดประตูหลังก่อนที่จะสาย

- Authentication vs Authorization — เข้าใจให้ชัด
- OAuth 2.0 / OpenID Connect overview
- Rate limiting, throttling — ป้องกัน abuse
- API key management & scoping
- CORS — เข้าใจจริงๆ ว่ามันป้องกันอะไร (และไม่ป้องกันอะไร)
- Common API vulnerabilities: BOLA, mass assignment, excessive data exposure
- **Life lesson:** ให้สิทธิ์เท่าที่จำเป็น — ทั้งในระบบและในชีวิต

### บทความที่ 16: Cloud Security — เมื่อ Infrastructure อยู่บน Cloud ความเสี่ยงก็ตามไปด้วย

- **Shared Responsibility Model** — อะไรที่ cloud provider ดูแล อะไรที่คุณต้องดูแลเอง
  - AWS / GCP / Azure ต่างกันยังไง (ภาพรวม)
- **IAM (Identity & Access Management)** — หัวใจของ cloud security
  - Principle of least privilege สำหรับ IAM roles & policies
  - อย่าใช้ root account / admin credentials ในงานประจำวัน
  - Service accounts & short-lived credentials
- ภัยที่พบบ่อยจากการ misconfigure:
  - S3 bucket / Cloud Storage เปิด public โดยไม่ตั้งใจ
  - Security groups / Firewall rules เปิดกว้างเกินไป
  - Database exposed to the internet
- **Infrastructure as Code (IaC) security** — scan Terraform, CloudFormation ก่อน deploy
  - Tools: checkov, tfsec, Bridgecrew
- Logging & monitoring บน cloud:
  - CloudTrail (AWS), Cloud Audit Logs (GCP)
  - ตั้ง alerts สำหรับ suspicious activities
- Container security basics: scan Docker images, ใช้ minimal base images, ไม่ run as root
- **Hands-on:** audit IAM permissions ของทีมคุณวันนี้
- **Life lesson:** "ย้ายบ้าน" ไม่ได้แปลว่าปลอดภัยขึ้นอัตโนมัติ — บ้านใหม่ก็ต้องล็อกประตู ติดกล้อง เหมือนกัน

---

## Part 6 — AI & LLM Security: ภัยยุคใหม่ที่ Dev ต้องรู้ทัน

### บทความที่ 17: AI & LLM Security — เมื่อ AI เป็นทั้งเครื่องมือและช่องโหว่

- AI/LLM กำลังเปลี่ยนวิธีทำงานของ dev — แต่มาพร้อมความเสี่ยงใหม่
- **ความเสี่ยงจากการใช้ AI ในงาน dev:**
  - Paste code / credentials / internal data เข้า ChatGPT, Copilot, Claude
  - ข้อมูลที่ส่งไป AI อาจถูกใช้ train model ต่อ (ขึ้นกับ policy ของ provider)
  - AI-generated code อาจมีช่องโหว่ — ห้าม trust โดยไม่ review
- **Prompt Injection** — ช่องโหว่ใหม่ของยุค LLM
  - Direct prompt injection: หลอก AI ให้ทำสิ่งที่ไม่ควรทำ
  - Indirect prompt injection: ซ่อนคำสั่งในข้อมูลที่ AI อ่าน
  - ตัวอย่างจริง: injection ผ่านอีเมล, เว็บ, เอกสารที่ AI ประมวลผล
- **Data leakage ผ่าน AI tools:**
  - Model inversion, training data extraction
  - วิธีใช้ AI tools อย่างปลอดภัย — อะไรส่งได้ อะไรห้ามส่ง
  - Company policy สำหรับ AI usage — ทำไมต้องมี
- **AI ในฝั่ง attacker:**
  - Deepfake voice/video สำหรับ social engineering
  - AI-generated phishing ที่แยกไม่ออกจากของจริง
  - Automated vulnerability scanning ด้วย AI
- **แนวทางใช้ AI อย่างปลอดภัยในทีม:**
  - กำหนด approved tools & tier (อะไรใช้ได้กับ data ระดับไหน)
  - ใช้ enterprise versions ที่มี data privacy guarantees
  - Review AI output เสมอ — AI เป็น assistant ไม่ใช่ authority
- **Life lesson:** เครื่องมือที่ทรงพลังที่สุดก็อันตรายที่สุดถ้าใช้ไม่เป็น — มีดคมหั่นผักได้ดี แต่ก็บาดมือได้ง่าย

---

## Part 7 — Data Privacy & Compliance: ปกป้องข้อมูล ปกป้องความเชื่อมั่น

### บทความที่ 18: Data Privacy & PDPA — สิ่งที่ Dev ไทยต้องรู้ตามกฎหมาย

- **PDPA (พ.ร.บ. คุ้มครองข้อมูลส่วนบุคคล)** — ภาพรวมที่ dev ต้องเข้าใจ
  - Personal data คืออะไรบ้าง (กว้างกว่าที่คิด — ชื่อ, email, IP address, cookie ID)
  - Sensitive data — ข้อมูลสุขภาพ, ศาสนา, ข้อมูลชีวภาพ ต้องระวังเป็นพิเศษ
  - สิทธิของเจ้าของข้อมูล: access, rectification, erasure, portability
- **GDPR** overview — ถ้าทีมทำ product ที่ให้บริการ EU users
- **Data classification** — จัดระดับข้อมูลก่อน จะได้ปกป้องถูกที่
  - Public → Internal → Confidential → Restricted
  - ข้อมูลแต่ละระดับต้องจัดการยังไง
- **Privacy by Design** — ออกแบบระบบให้ protect privacy ตั้งแต่แรก
  - Data minimization — เก็บเท่าที่จำเป็น
  - Purpose limitation — ใช้ตามวัตถุประสงค์ที่แจ้งไว้
  - Consent management — ขอ consent อย่างถูกต้อง
- สิ่งที่ dev ต้องระวังในงานประจำวัน:
  - อย่า log PII โดยไม่จำเป็น
  - Anonymization & pseudonymization
  - Data retention — ลบข้อมูลเมื่อหมดความจำเป็น
  - Third-party data sharing — ส่งข้อมูลให้ vendor ต้องมี DPA
- โทษตาม PDPA — ปรับสูงสุด 5 ล้านบาท + โทษทางอาญา
- **Life lesson:** ข้อมูลส่วนบุคคลของคนอื่นไม่ใช่ของเรา — เราแค่ได้รับความไว้วางใจให้ดูแล การรักษาความไว้ใจเป็นหลักการที่ใช้ได้ทุกความสัมพันธ์

---

## Part 8 — Advanced Concepts: คิดแบบ Security Professional

### บทความที่ 19: Zero Trust — อย่าเชื่อใจใคร แม้แต่คนในเครือข่ายเดียวกัน

- Traditional security model: castle-and-moat (ข้างในปลอดภัย ข้างนอกอันตราย)
- ทำไม model เดิมใช้ไม่ได้อีกแล้ว (remote work, cloud, BYOD)
- **Zero Trust Architecture** — "Never trust, always verify"
- หลักการสำคัญ:
  - Verify explicitly (ทุก request ต้อง authenticate + authorize)
  - Least privilege access
  - Assume breach
- ตัวอย่างการนำไปใช้จริง: BeyondCorp (Google), micro-segmentation, identity-aware proxy
- Zero Trust สำหรับ dev team: SSH, VPN, database access ควรจัดการยังไง
- **Life lesson:** "อยู่ในกลุ่มเดียวกัน" ไม่ได้แปลว่าปลอดภัย — ตรวจสอบทุกครั้ง ทุกคน ทุกสถานการณ์

### บทความที่ 20: Threat Modeling — คิดแบบ Attacker เพื่อป้องกันแบบ Defender

- Threat modeling คืออะไร — การคิดล่วงหน้าว่า "อะไรอาจพังได้"
- Framework ที่ใช้ได้: STRIDE, DREAD, Attack Trees
- วิธีทำ threat model สำหรับ feature ใหม่:
  - ระบุ assets (อะไรที่ต้องปกป้อง)
  - ระบุ threat actors (ใครอาจโจมตี)
  - ระบุ attack vectors (โจมตีทางไหน)
  - ระบุ mitigations (ป้องกันยังไง)
- รวม threat modeling เข้ากับ development process (shift-left security)
- **Life lesson:** ก่อนตัดสินใจเรื่องสำคัญ ลองคิดว่า "อะไรอาจผิดพลาดได้" — เป็น mental model ที่ใช้ได้ทุกเรื่อง

### บทความที่ 21: Incident Response — เมื่อโดน Hack แล้ว ทำอะไรต่อ

- ทำไมต้องมี incident response plan — ตื่นตระหนกคือศัตรูตัวจริง
- ขั้นตอนพื้นฐาน: Identify → Contain → Eradicate → Recover → Lessons Learned
- สำหรับ dev team:
  - Credential rotation ทันที
  - ตรวจ audit logs
  - แจ้ง stakeholders
  - Post-mortem without blame
- Tabletop exercises — ซ้อมรับมือเหตุการณ์เหมือนซ้อมหนีไฟ
- **Life lesson:** เตรียมแผนสำหรับสิ่งที่ไม่อยากให้เกิด — วิกฤตไม่น่ากลัวเท่าการไม่มีแผนรับมือ

---

## Part 9 — Building Security Culture: สร้างวัฒนธรรมความปลอดภัยในทีม

### บทความที่ 22: Security Culture — ทำยังไงให้ทั้งทีมใส่ใจ ไม่ใช่แค่คนเดียว

- ทำไม security ต้องเป็นเรื่องของทุกคน ไม่ใช่แค่ security team
- สร้าง "security champions" ในทีม
- Blameless post-mortems — ส่งเสริมให้คนกล้ารายงานปัญหา
- ทำ security เป็นส่วนหนึ่งของ Definition of Done
- Security awareness activities: CTF, phishing simulation, lunch & learn
- **Life lesson:** วัฒนธรรมที่ดีสร้างจากการทำให้เป็นเรื่องปกติ ไม่ใช่เรื่องน่ากลัว

### บทความที่ 23: Personal Security Checklist — สรุปทุกอย่าง ลงมือทำเลย

- **Checklist รวม** สำหรับทุกคนในทีม:
  - Password & Authentication checklist
  - Device security checklist
  - Developer security checklist
  - Cloud & Infrastructure checklist
  - AI tools usage checklist
  - Data privacy checklist
  - Backup & Recovery checklist
  - Online hygiene checklist
- Self-assessment: คุณอยู่ level ไหนของ security awareness
- แผน 30 วันสำหรับยกระดับ security posture ส่วนตัว
- แหล่งเรียนรู้เพิ่มเติม: blogs, podcasts, courses ที่แนะนำ
- **Life lesson:** ความรู้ที่ไม่ลงมือทำ ไม่มีค่า — เริ่มจากสิ่งเล็กๆ วันนี้

---

## สรุปโครงสร้าง

| Part | หัวข้อ | จำนวนบทความ |
|------|--------|-------------|
| 1 | Foundation: Mindset | 1 |
| 2 | Password & Authentication | 4 |
| 3 | Cryptography พื้นฐาน | 1 |
| 4 | Everyday Threats | 5 |
| 5 | Developer-Specific | 5 |
| 6 | AI & LLM Security | 1 |
| 7 | Data Privacy & Compliance | 1 |
| 8 | Advanced Concepts | 3 |
| 9 | Security Culture & Action | 2 |
| **รวม** | | **23 บทความ** |

---

## แนวทางการเขียนแต่ละบทความ

- **ความยาว:** ประมาณ 1,500–2,500 คำต่อบทความ (อ่านจบใน 10–15 นาที)
- **โครงสร้าง:** เปิดด้วยเรื่องราว/กรณีศึกษา → อธิบายแนวคิด → Hands-on/Action items → ปิดด้วย Life lesson
- **ทุกบทความจบด้วย:**
  - **Action Items** — 3-5 สิ่งที่ทำได้ทันที
  - **Life Lesson** — เชื่อมโยง cybersecurity กับชีวิตจริง
  - **อ่านต่อ** — link ไปบทความถัดไปใน series

---

## ลำดับการอ่านแนะนำ

```
สำหรับทุกคน (ไม่ว่าจะเป็น dev หรือไม่):
  บทความ 1 → 2 → 3 → 4 → 5 → 7 → 8 → 9 → 10 → 11 → 23

เพิ่มเติมสำหรับ dev:
  บทความ 6 → 12 → 13 → 14 → 15 → 16 → 17 → 18

สำหรับ tech lead / architect:
  บทความ 19 → 20 → 21 → 22
```

---

*Outline v2.0 — เพิ่ม Cryptography, Ransomware, Backup & DR, Cloud Security, AI/LLM Security, Data Privacy & PDPA*
*พร้อมสำหรับ review และเริ่มเขียนบทความแรก*
