# บทความที่ 18: Data Privacy & PDPA — สิ่งที่ Dev ไทยต้องรู้ตามกฎหมาย

**CyberSecurity Awareness Series — Part 7: Data Privacy & Compliance**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของบริษัทที่ "โดนปรับ 7 ล้านบาท เพราะลืมตั้งค่า cookie consent"

เดือนสิงหาคม ปี 2024 — **คณะกรรมการคุ้มครองข้อมูลส่วนบุคคล (สคส.)** ประกาศคำสั่งปรับเงิน บริษัทเอกชนแห่งหนึ่ง **มูลค่า 7 ล้านบาท** จากการละเมิด **PDPA**

สาเหตุของการถูกปรับ:

1. **เก็บข้อมูลส่วนบุคคล** ของลูกค้าโดยไม่ขอ consent
2. **ใช้ cookies tracking** โดยไม่แจ้ง
3. **ส่งข้อมูล** ให้บริษัท third-party (ad network) โดยไม่ได้รับอนุญาต
4. **ไม่มี privacy policy** ที่ชัดเจน
5. **เมื่อมี data subject request** (user ขอเข้าถึงข้อมูลของตัวเอง) — ไม่ตอบกลับภายใน 30 วันที่กฎหมายกำหนด

นี่คือเคสแรกๆ ที่ สคส. **บังคับใช้กฎหมายเต็มรูป** ในประเทศไทย — และเป็นสัญญาณว่าบริษัททุกแห่งต้องเริ่มจริงจังแล้ว

[อ่านข่าวเพิ่มเติม](https://www.pdpc.or.th)

ที่น่าตกใจคือ — **ค่าปรับสูงสุดของ PDPA = 5 ล้านบาท ต่อกรณี + โทษทางอาญา (จำคุกถึง 6 เดือน)** — ส่วนใหญ่ของ developer ไม่รู้ว่าตัวเองอาจ **รับผิดส่วนตัว** ถ้าทำผิด

ในยุคปี 2026 — กฎหมายข้อมูลส่วนบุคคลคือ **เรื่องที่ dev ทุกคน** ต้องเข้าใจ ไม่ใช่ "เรื่องของฝ่ายกฎหมาย"

> **คุณเขียน code ที่จัดการข้อมูลของคนอื่น = คุณรับผิดชอบตามกฎหมาย**

ในบทความนี้เราจะดู:

1. **PDPA** (พ.ร.บ. คุ้มครองข้อมูลส่วนบุคคล) — ที่ dev ไทยต้องเข้าใจ
2. **GDPR** — สำหรับ product ที่ให้บริการ EU users
3. **Data Classification** — จัดระดับข้อมูล
4. **Privacy by Design** — ออกแบบให้ปกป้องตั้งแต่แรก
5. **สิ่งที่ Dev ต้องระวัง** ในงานประจำวัน
6. **โทษ + Compliance**

---

## PDPA คืออะไร — ภาพรวมสำหรับ Dev

**PDPA** = พระราชบัญญัติคุ้มครองข้อมูลส่วนบุคคล พ.ศ. 2562 (Personal Data Protection Act 2019)

- **ประกาศ**: 27 พฤษภาคม 2562
- **บังคับใช้เต็ม**: 1 มิถุนายน 2565 (เลื่อนจาก 2563 เพราะ COVID)
- **องค์กรกำกับ**: คณะกรรมการคุ้มครองข้อมูลส่วนบุคคล (สคส. / PDPC)

PDPA ไทย **ใกล้เคียง GDPR** ของ EU — ทั้งโครงสร้างและหลักการ ดังนั้นถ้าเข้าใจ PDPA จะเข้าใจ GDPR ได้ง่าย

### ใครต้องปฏิบัติตาม PDPA

PDPA ใช้กับ:

1. **บริษัท / องค์กร / บุคคลในไทย** ที่เก็บ-ใช้-เปิดเผยข้อมูลส่วนบุคคล
2. **บริษัทต่างประเทศ** ที่:
   - มีสาขา / กิจการในไทย
   - ให้บริการคนไทย
   - Monitor พฤติกรรมคนไทย

→ **ครอบคลุมแทบทุกธุรกิจ** ที่เกี่ยวกับข้อมูลคนไทย

### Personal Data คืออะไร

นี่คือจุดที่ dev เข้าใจผิดบ่อยที่สุด — **personal data กว้างกว่าที่คิด**:

#### ข้อมูลที่ "ระบุตัวตน" ได้โดยตรง

- ชื่อ, นามสกุล
- เลขบัตรประชาชน
- เลข passport
- เลข driver's license
- รูปถ่ายใบหน้า

#### ข้อมูลที่ "ระบุตัวตน" ได้โดยอ้อม

- Email address
- เบอร์โทรศัพท์
- ที่อยู่
- Username
- **IP address** (!) — ในหลาย jurisdiction ถือเป็น personal data
- **Cookie ID** (!) — ใน GDPR ถือเป็น personal data
- **Device ID** (UDID, Android ID, IDFA)
- **Location data** (GPS coordinates)
- Browser fingerprint

#### Sensitive Personal Data — ระวังเป็นพิเศษ

ตาม PDPA Section 26 — ข้อมูลที่ต้องการ **explicit consent** หรือมี legal basis เฉพาะ:

- **เชื้อชาติ, ศาสนา, ความเชื่อ**
- **พฤติกรรมทางเพศ**
- **ประวัติอาชญากรรม**
- **ข้อมูลสุขภาพ + ความพิการ**
- **ข้อมูลพันธุกรรม**
- **ข้อมูลชีวภาพ** (biometric — fingerprint, face recognition data)
- **ข้อมูลทางการเงิน** (ไม่ระบุชัดเจน แต่ระวัง)
- **ข้อมูลสมาชิกสหภาพแรงงาน**

> **กฎ: ถ้าข้อมูลที่คุณเก็บสามารถ link กลับไปหา "ตัวตน" ของบุคคลได้ — = personal data**

### หลักการของ PDPA — 7 หลักหลัก

#### 1. Lawfulness, Fairness, Transparency

ต้องประมวลผลข้อมูลโดย:
- **มีฐานทางกฎหมาย** (consent, contract, legal obligation, vital interest, public interest, legitimate interest)
- **เป็นธรรม** ต่อ data subject
- **โปร่งใส** — บอกว่าทำอะไรกับข้อมูล

#### 2. Purpose Limitation

เก็บข้อมูลเพื่อจุดประสงค์ที่ระบุไว้ชัดเจน — **ห้าม** เอาไปใช้นอกจากนั้นโดยไม่ขอใหม่

```
✅ "เก็บ email เพื่อส่ง newsletter"
❌ ใช้ email นั้นไปขายต่อให้บริษัทอื่น (ไม่ได้ระบุไว้)
```

#### 3. Data Minimization

เก็บ **เท่าที่จำเป็น**:

```python
# ❌ Over-collection
class UserProfile:
    name: str
    email: str
    phone: str
    address: str
    date_of_birth: str
    gender: str
    income: int      # ทำไมต้องเก็บ?
    marital_status: str  # ทำไมต้องเก็บ?
    ...

# ✅ Minimum necessary
class UserProfile:
    email: str       # for login
    name: str        # for display
```

#### 4. Accuracy

ข้อมูลต้อง **ถูกต้อง** + ทันสมัย + แก้ไขได้

→ User ต้องสามารถ update ข้อมูลตัวเองได้

#### 5. Storage Limitation

เก็บ **ไม่นานเกินจำเป็น**:

```python
# ✅ Retention policy
- Active user data: keep while account is active
- Inactive >1 year: anonymize
- Inactive >2 years: delete
- Transaction records: 7 years (per accounting law)
- Logs: 90 days
```

#### 6. Integrity & Confidentiality (Security)

ปกป้องด้วย technical + organizational measures:
- Encryption (at rest + in transit)
- Access control
- Authentication
- Audit logs

#### 7. Accountability

**Demonstrate compliance** ได้:
- Documented policies
- Records of processing
- Training records
- Audit trails

### สิทธิของ Data Subject (เจ้าของข้อมูล) — 8 สิทธิ

User มีสิทธิ์ทำ 8 อย่างกับข้อมูลของตน:

#### 1. Right to be Informed

User ต้องรู้ว่าคุณ **เก็บอะไร, เพื่อจุดประสงค์อะไร, ใช้กับใคร**

→ Privacy notice / privacy policy ที่ชัดเจน

#### 2. Right to Access

User ขอดูข้อมูลของตัวเองได้:

```python
@app.route('/api/me/data', methods=['GET'])
@login_required
def export_my_data():
    user = current_user
    data = {
        "profile": user.profile,
        "orders": list(user.orders),
        "logs": user.activity_logs,
        # ... ทุกอย่างที่เกี่ยวข้องกับ user
    }
    return jsonify(data)
```

#### 3. Right to Rectification

User ขอแก้ไขข้อมูลที่ผิดได้

#### 4. Right to Erasure (Right to be Forgotten)

User ขอลบข้อมูลของตัวเองได้:

```python
@app.route('/api/me/account', methods=['DELETE'])
@login_required
def delete_my_account():
    user = current_user
    
    # Anonymize ข้อมูลที่ต้องเก็บ (เช่น orders for accounting)
    anonymize_orders(user.id)
    
    # Delete ข้อมูล personal
    delete_personal_data(user.id)
    
    # Log การลบ (without PII)
    log.info("account_deleted", extra={"user_id_hash": hash(user.id)})
```

**ข้อยกเว้น:** ถ้ามี legal obligation ให้เก็บต่อ (เช่น tax records 7 ปี) — ไม่ต้องลบ

#### 5. Right to Restrict Processing

User ขอ "หยุดประมวลผล" ได้ — ข้อมูลยังเก็บไว้ แต่ห้ามใช้

#### 6. Right to Data Portability

User ขอ data ของตัวเองในรูปแบบที่ "อ่านได้ + portable":

```python
@app.route('/api/me/export', methods=['GET'])
@login_required
def portable_export():
    return jsonify({
        "format_version": "1.0",
        "exported_at": datetime.utcnow().isoformat(),
        "data": user_to_machine_readable(current_user)
    })
```

→ User เอาไปใช้กับ service อื่นได้

#### 7. Right to Object

User ขอ "คัดค้าน" การใช้ข้อมูลของตัวเองได้ (โดยเฉพาะสำหรับ direct marketing)

#### 8. Right Not to be Subject to Automated Decision-Making

User ขอ **ไม่ถูกตัดสินอัตโนมัติ** ได้ (เช่น AI deny loan โดยไม่มี human review)

→ ต้องมี **human-in-the-loop** สำหรับ high-stakes decisions

---

## GDPR — สำหรับ Product ที่ให้บริการ EU Users

ถ้า product ของคุณ **ให้บริการ EU users** (แม้บริษัทอยู่ไทย) — GDPR applies

### ความเหมือน-ต่างกับ PDPA

| Aspect | GDPR (EU) | PDPA (Thailand) |
|---|---|---|
| Scope | EU residents | Thai residents |
| Effective | 2018 | 2022 |
| Max fine | €20M หรือ 4% global revenue | 5M THB ต่อกรณี |
| Sensitive data | ใกล้เคียงกัน | ใกล้เคียงกัน |
| Data Subject Rights | 8 rights (เหมือนกัน) | 8 rights |
| Breach notification | 72 hours | 72 hours (ใน PDPA Section 37) |
| DPO required | บางกรณี | บางกรณี |

### When GDPR Applies

GDPR ใช้กับธุรกิจที่:

1. **มีสาขาใน EU** หรือ
2. **ให้บริการ EU users** แม้ไม่มีสาขา (เช่น website ภาษา EU + ราคา EU + รับ EU customers)
3. **Monitor EU users** (เช่น behavioral tracking)

### GDPR Quick Compliance Checklist

- [ ] **Privacy notice** ใน language ของ user
- [ ] **Consent mechanism** ที่ explicit + opt-in
- [ ] **Cookie banner** ที่ compliant
- [ ] **DSAR (Data Subject Access Request)** process
- [ ] **DPA (Data Processing Agreement)** กับ vendors
- [ ] **DPO** (Data Protection Officer) ถ้ามีการประมวลผล large scale
- [ ] **EU representative** ถ้าไม่มีสาขาใน EU
- [ ] **Data transfer safeguards** ถ้าส่ง data ออก EU (SCC, BCR)
- [ ] **Breach notification** procedure (72 hours)

---

## Data Classification — จัดระดับก่อนปกป้อง

ก่อนปกป้องได้ — ต้องรู้ว่ามีอะไร และระดับไหน

### 4-Tier Classification (Standard)

#### Tier 1: Public

ข้อมูลที่เปิดเผยได้ทั่วไป:
- Marketing materials
- Public website content
- Published press releases
- Open source code

→ Protection: minimal (just integrity)

#### Tier 2: Internal

ข้อมูลที่ใช้ภายใน — share ภายในได้:
- Internal documents
- Employee directory
- Org charts
- Non-customer code

→ Protection: access control, internal sharing OK

#### Tier 3: Confidential

ข้อมูลที่ sensitive:
- Customer email lists
- Internal financials (non-public)
- Strategic plans
- Customer support transcripts

→ Protection: encryption, RBAC, audit logs

#### Tier 4: Restricted / Highly Sensitive

ข้อมูลที่ critical:
- Customer PII (full name + sensitive)
- Healthcare records (PHI)
- Payment card data (PCI)
- Authentication credentials
- Trade secrets, M&A info

→ Protection: 
- Encryption with HSM-backed keys
- Strong access control + MFA
- Detailed audit logs
- DLP monitoring
- Regular audits

### How to Classify

#### Step 1: Inventory ข้อมูลทั้งหมด

```yaml
data_inventory:
  - name: user_profiles
    fields: [name, email, phone, address]
    location: postgres.users
    classification: Tier 3 (Confidential)
    retention: 90 days after account closure
  
  - name: order_history
    fields: [order_id, items, total, date]
    location: postgres.orders
    classification: Tier 3 (Confidential) + Tier 4 (financial)
    retention: 7 years (accounting)
    
  - name: medical_records
    fields: [diagnosis, treatments, prescriptions]
    location: postgres.medical
    classification: Tier 4 (Restricted, PHI)
    retention: 25 years (per healthcare law)
```

#### Step 2: Tag ข้อมูลใน System

```sql
-- PostgreSQL columns with comment
COMMENT ON COLUMN users.email IS 'classification:tier3,pii';
COMMENT ON COLUMN users.medical_record IS 'classification:tier4,phi';
```

#### Step 3: Apply Controls ตาม Tier

```yaml
Tier 4 controls:
  - Encryption: AES-256 with KMS
  - Access: RBAC + MFA + just-in-time
  - Audit: every read/write logged
  - Backup: encrypted, separate location
  - Retention: per law
```

---

## Privacy by Design — ออกแบบให้ Privacy ตั้งแต่แรก

**Privacy by Design** เป็นหลัก 7 ข้อที่ Ann Cavoukian พัฒนา (1990s) — ปัจจุบันเป็น **GDPR Article 25** + อยู่ใน PDPA

### 7 Foundational Principles

#### 1. Proactive not Reactive

ป้องกันก่อนเกิด ไม่ใช่ "wait and see"

#### 2. Privacy as Default

ระบบต้องปลอดภัย **by default** — user ไม่ต้องไป opt-in:

```python
# ❌ Default = share data
class UserSettings:
    allow_marketing_emails = True   # ❌ default opt-in
    allow_third_party_share = True  # ❌ default opt-in

# ✅ Default = don't share
class UserSettings:
    allow_marketing_emails = False  # ✅ user must opt-in
    allow_third_party_share = False # ✅ default safest
```

#### 3. Privacy Embedded into Design

Privacy เป็น core feature — ไม่ใช่ bolt-on

#### 4. Full Functionality (Positive-Sum)

Privacy + functionality ไปด้วยกันได้ — ไม่ใช่ trade-off

#### 5. End-to-End Security

Encrypt at rest + in transit, ทุก step

#### 6. Visibility & Transparency

User เห็นได้ว่าระบบทำอะไร — privacy policy, data dashboard

#### 7. Respect for User Privacy

User-centric — สิทธิ์ของ user มาก่อน convenience ของ business

### Practical Implementation

#### Data Minimization in Practice

```python
# ❌ เก็บทุกอย่างเผื่อ
@dataclass
class UserSignup:
    name: str
    email: str
    phone: str
    date_of_birth: str
    gender: str
    address: str
    occupation: str
    income: int

# ✅ เก็บเท่าที่ต้องตอน signup
@dataclass  
class UserSignup:
    email: str
    password: str

# เก็บเพิ่มเฉพาะตอนต้องใช้:
# - Phone: ตอน setup 2FA
# - Address: ตอนสั่งของ
# - DOB: ตอนต้องตรวจอายุ (กฎหมาย)
```

#### Pseudonymization

แทนที่จะเก็บ "Name + Email" — ใช้ **pseudonym** + key separate:

```python
# ❌
user_data = {
    "id": "tanin@example.com",  # email = identifier
    "actions": [...]
}

# ✅ Pseudonymize
user_data = {
    "id": "user_a8f3k2",  # pseudonym (random)
    "actions": [...]
}

# Mapping ใน secure separate store:
mapping = {
    "user_a8f3k2": "tanin@example.com"
}
```

→ ถ้า data leak — ผู้โจมตียัง map กลับไม่ได้ ถ้าไม่มี mapping

#### Anonymization

ลบ identifier ทั้งหมด — **ย้อนกลับไม่ได้**:

```python
# Original
{
    "name": "Tanin T.",
    "email": "tanin@example.com",
    "age": 35,
    "city": "Bangkok",
    "purchases": ["coffee", "book"]
}

# Anonymized
{
    "age_range": "30-39",
    "city": "Bangkok",
    "purchases": ["coffee", "book"]
}
```

⚠️ **Anonymization is hard** — Netflix และ AOL เคยปล่อย "anonymized" data แต่ researcher de-anonymize ได้ผ่าน correlation

→ ใช้ **k-anonymity, differential privacy** — มาตรฐานทางคณิตศาสตร์

#### Consent Management

```python
@app.route('/signup', methods=['POST'])
def signup():
    data = request.json
    
    # ✅ Granular consent
    user = User.create(
        email=data['email'],
        password_hash=hash_password(data['password']),
        consents={
            "terms_of_service": data['accepted_tos'],
            "privacy_policy": data['accepted_privacy'],
            "marketing_emails": data.get('marketing_consent', False),
            "analytics": data.get('analytics_consent', False),
            "third_party_sharing": data.get('third_party_consent', False),
        },
        consent_timestamp=datetime.utcnow(),
        consent_ip=request.remote_addr,
    )
```

→ Track consent ละเอียด — สามารถพิสูจน์ได้ตอน audit

#### Cookie Consent

```html
<!-- ❌ Pre-checked -->
<input type="checkbox" checked> Accept all cookies

<!-- ✅ User must actively check -->
<input type="checkbox"> Functional cookies
<input type="checkbox"> Analytics cookies
<input type="checkbox"> Marketing cookies

<!-- ✅ Equal prominence -->
[Accept All] [Reject All] [Manage]
```

หลายเว็บใช้ "dark pattern" ที่ Accept ใหญ่ Reject เล็ก — **violation** ของ GDPR + PDPA

---

## สิ่งที่ Dev ต้องระวังในงานประจำวัน

### 1. อย่า Log PII โดยไม่จำเป็น

```python
# ❌
log.info(f"User {user.email} placed order {order.id}")
log.error(f"Failed to process payment for {user.full_name}, card {user.card_number}")

# ✅
log.info("order_placed", extra={
    "user_id": user.id,         # internal ID
    "order_id": order.id
})
log.error("payment_failed", extra={
    "user_id": user.id,
    "card_last_4": user.card_number[-4:],
    "error_code": "..."
})
```

#### Logs Never to Capture

- ❌ Full credit card number
- ❌ CVV
- ❌ Password (even hashed)
- ❌ Authentication tokens (full)
- ❌ SSN / National ID (full)
- ❌ Health data
- ❌ Plaintext email body content
- ❌ Document content

### 2. Database Schema with Privacy

```sql
-- ❌ ทุก field ในตาราง users
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    name VARCHAR(255),
    phone VARCHAR(20),
    ssn VARCHAR(20),
    medical_history TEXT,
    payment_card VARCHAR(20)
);

-- ✅ แยก sensitive ออก + encrypt
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),       -- needed for login
    name VARCHAR(255)         -- needed for display
);

CREATE TABLE user_sensitive (
    user_id INT REFERENCES users(id),
    ssn_encrypted BYTEA,      -- encrypted with KMS
    medical_history_encrypted BYTEA
);

-- Different access controls
GRANT SELECT ON users TO app_user;
GRANT SELECT ON user_sensitive TO admin_user_only;
```

### 3. Anonymize Test Data

```python
# ❌ Copy production → staging
INSERT INTO staging.users SELECT * FROM production.users;
# 💥 Production PII ใน staging

# ✅ Anonymize ก่อน copy
INSERT INTO staging.users (id, email, name)
SELECT 
    id,
    'user' || id || '@test.com',                           -- fake email
    'Test User ' || id                                      -- fake name
FROM production.users;
```

### 4. Data Retention

```python
# Daily job
def cleanup_old_data():
    # Inactive accounts > 2 years
    inactive = User.query.filter(
        User.last_login < datetime.utcnow() - timedelta(days=730)
    ).all()
    for user in inactive:
        anonymize_user(user)
    
    # Logs > 90 days
    Log.query.filter(
        Log.created_at < datetime.utcnow() - timedelta(days=90)
    ).delete()
    
    # Carts ที่ abandoned > 30 days
    AbandonedCart.query.filter(
        AbandonedCart.created_at < datetime.utcnow() - timedelta(days=30)
    ).delete()
    
    db.session.commit()
```

### 5. Third-party Data Sharing

ก่อน share ข้อมูลกับ vendor (analytics, payment, email) — ต้องมี:

#### Data Processing Agreement (DPA)

Contract ที่:
- ระบุ data ที่ส่ง
- ระบุ purpose
- ระบุ security measures
- ห้าม sub-process โดยไม่อนุญาต
- กำหนด retention + deletion

#### Vendor Security Assessment

ตรวจ:
- SOC 2 / ISO 27001 certification
- Privacy policy
- Breach history
- Security practices

#### Data Transfer Safeguards (สำหรับ international transfer)

ส่ง data ไป US / non-EU country:
- **SCC** (Standard Contractual Clauses) — EU template
- **BCR** (Binding Corporate Rules)
- **Adequacy Decision** — country ที่ EU/Thai ยอมรับ

ในไทย — สคส. ยังไม่ออกรายละเอียดเรื่อง international transfer ชัดเจน แต่ใกล้เคียง GDPR

---

## โทษตาม PDPA

### Civil Penalties

```yaml
ระดับ 1 (น้อยที่สุด):
  - ค่าปรับสูงสุด: 1 ล้านบาท
  - กรณี: ไม่แจ้ง data subject เรื่อง processing

ระดับ 2:
  - ค่าปรับสูงสุด: 3 ล้านบาท  
  - กรณี: เก็บ-ใช้ข้อมูลเกินวัตถุประสงค์

ระดับ 3 (ร้ายแรงที่สุด):
  - ค่าปรับสูงสุด: 5 ล้านบาท
  - กรณี: ประมวลผล sensitive data ผิด, transfer ออกประเทศโดยไม่ป้องกัน
```

### Criminal Penalties (PDPA Section 79)

- **เปิดเผย** ข้อมูลส่วนบุคคลโดยไม่ชอบ → จำคุก **ไม่เกิน 6 เดือน** + ปรับไม่เกิน **500,000 บาท**
- ถ้าเป็น sensitive data → จำคุกไม่เกิน **1 ปี** + ปรับ **1 ล้าน**
- ถ้าทำเพื่อหวังผลประโยชน์ → จำคุกไม่เกิน **1 ปี** + ปรับ **1 ล้าน**

### Compensation to Data Subjects

User ที่เสียหายฟ้องคดีแพ่งได้ — **ไม่จำกัดวงเงิน** (แล้วแต่ความเสียหายจริง)

### ใครรับผิด

#### Data Controller (ผู้ควบคุมข้อมูล)

บริษัท / องค์กรที่ตัดสินใจว่าจะเก็บ-ใช้ข้อมูลยังไง — รับผิดเป็นหลัก

#### Data Processor (ผู้ประมวลผล)

Vendor ที่ประมวลผลข้อมูลให้ controller — รับผิดในส่วนของตน

#### พนักงาน / Director

อาจรับผิดส่วนตัวถ้า:
- ฝ่าฝืน knowingly
- Director ที่ไม่ดูแลให้ compliance

---

## Hands-on: Audit Privacy Compliance

### Step 1: Data Mapping

```python
# Audit ทุก data store
data_inventory = []

# Database tables
for table in db.tables:
    columns = analyze_columns(table)
    pii_columns = [c for c in columns if is_pii(c)]
    data_inventory.append({
        "location": f"db.{table}",
        "pii_columns": pii_columns,
        "classification": classify(pii_columns),
        "retention_policy": ?,  # ต้องตอบ
        "access_control": ?,
    })

# Logs
# Files (S3, GCS)
# Queues
# Cache
# Backups
# Third-party services
```

### Step 2: Privacy Notice Review

```markdown
# Privacy Notice Checklist

- [ ] บอก identity ของ controller ชัดเจน
- [ ] List ประเภทข้อมูลที่เก็บ
- [ ] List วัตถุประสงค์ของการเก็บ
- [ ] List ฐานทางกฎหมาย (consent, contract, etc.)
- [ ] บอกว่าจะเก็บนานแค่ไหน
- [ ] บอก third parties ที่ share ด้วย
- [ ] บอกสิทธิ์ของ user
- [ ] บอกวิธี contact (DPO email)
- [ ] อัปเดตล่าสุดเมื่อไหร่
```

### Step 3: Data Subject Request Process

```python
# DSAR endpoint
@app.route('/dsar', methods=['POST'])
def submit_dsar():
    request_type = request.json['type']  # access/rectify/erase/etc.
    user_email = request.json['email']
    
    # Verify user identity (สำคัญ!)
    verify_user(user_email)
    
    # Create ticket
    ticket = DSARTicket.create(
        type=request_type,
        email=user_email,
        deadline=datetime.utcnow() + timedelta(days=30)  # PDPA: 30 days
    )
    
    # Notify privacy team
    notify_team(ticket)
    
    return jsonify({
        "ticket_id": ticket.id,
        "expected_response": ticket.deadline.isoformat()
    })
```

→ ต้องตอบใน **30 วัน** (ขยายได้ 60 วันถ้า complex)

### Step 4: Breach Response

ถ้า data breach เกิดขึ้น:

1. **ภายใน 24 ชม.** — internal team know + start investigation
2. **ภายใน 72 ชม.** — notify สคส. ถ้า:
   - ข้อมูลที่รั่วเป็น sensitive
   - มีผลกระทบต่อ user
3. **Notify users** ถ้า "high risk" ต่อสิทธิ์ของ user
4. **Document** ทุกอย่าง

```python
# Breach response runbook
def handle_breach(severity, data_affected, count):
    # 1. Contain
    isolate_affected_systems()
    
    # 2. Assess
    assessment = {
        "data_type": data_affected,
        "count": count,
        "severity": severity,
        "risk_to_users": evaluate_risk()
    }
    
    # 3. Notify (within 72 hours if required)
    if requires_pdpc_notification(assessment):
        notify_pdpc(assessment)  # คณะกรรมการ PDPA
    
    if requires_user_notification(assessment):
        notify_affected_users(assessment)
    
    # 4. Document
    create_incident_report(assessment)
    
    # 5. Remediate
    fix_root_cause()
    review_controls()
```

---

## Action Items

### วันนี้

- [ ] **Audit codebase** — มีที่ไหน log PII ไหม? ลบ
- [ ] **Review database schema** — มี sensitive field ที่ไม่ encrypted ไหม?
- [ ] **Privacy notice** — มีและ updated ไหม?
- [ ] **Cookie banner** — compliant ไหม (no pre-check)?

### สัปดาห์นี้

- [ ] **Data mapping** — list ทุกแหล่งของ PII
- [ ] **Retention policy** — เขียนเป็นเอกสาร + automate cleanup
- [ ] **Vendor list** — มี DPA กับทุก vendor ที่ touch PII?
- [ ] **DSAR process** — มี endpoint + workflow

### เดือนนี้

- [ ] **Privacy by Design review** — ใน every new feature
- [ ] **Pseudonymization** — implement สำหรับ analytics
- [ ] **Encryption audit** — at-rest + in-transit ครบไหม
- [ ] **Test data anonymization** — staging ใช้ production data ไหม

### สำหรับ Tech Lead

- [ ] **DPO** appointment ถ้าต้องการ
- [ ] **Privacy training** สำหรับทีมทุก quarter
- [ ] **Incident response plan** สำหรับ breach
- [ ] **External privacy audit** ทุกปี

### สำหรับ CTO / Founders

- [ ] **PDPA compliance roadmap**
- [ ] **Cyber insurance** with PDPA coverage
- [ ] **Legal counsel** ที่เข้าใจ data protection
- [ ] **Budget for privacy infrastructure**

---

## บทเรียนชีวิตจากบทความนี้

> **ข้อมูลส่วนบุคคลของคนอื่นไม่ใช่ของเรา — เราแค่ได้รับความไว้วางใจให้ดูแล การรักษาความไว้ใจเป็นหลักการที่ใช้ได้ทุกความสัมพันธ์**

หลักการของ **fiduciary duty** (หน้าที่ดูแลความไว้ใจ) ใช้ได้ทุกที่ในชีวิต:

#### ในการเงิน

- ที่ปรึกษาทางการเงิน — ดูแลเงินของลูกค้า
- พนักงานธนาคาร — ดูแลเงินที่ฝาก
- กรรมการบริษัท — ดูแลผลประโยชน์ของผู้ถือหุ้น

#### ในการแพทย์

- หมอ — ดูแลข้อมูลคนไข้ (ไม่บอกใคร)
- พยาบาล — ดูแลความเป็นส่วนตัวของผู้ป่วย

#### ในกฎหมาย

- ทนาย — ดูแล attorney-client privilege

#### ในความสัมพันธ์

- เพื่อนสนิท — เก็บความลับของอีกคน
- ครอบครัว — เก็บเรื่องในครอบครัวไว้
- พี่น้อง — รู้เรื่องส่วนตัวของกันและกัน
- คู่รัก — รู้ความลับลึกที่สุด

> **ความไว้ใจ คือสกุลเงินที่มีค่าที่สุดในความสัมพันธ์ — เสียครั้งเดียว สร้างใหม่ยาก**

ในประเด็น data privacy:

- User trust คุณ → กรอก email, address, payment info
- คุณ leak → trust แตก
- ครั้งเดียว → user ไม่กลับมา

หลักการเดียวกับชีวิตจริง:

- **Reputation is built drop by drop, lost in a flood**
- ทำดี 1,000 ครั้ง — คนอาจจำได้น้อย
- ทำพลาด 1 ครั้ง (ใหญ่) — คนจำได้ตลอด

> **Privacy ไม่ใช่แค่ compliance — เป็น respect**

เมื่อคุณ:
- เก็บ data เกินจำเป็น = แสดงว่าคุณคิดว่า "ของของ user เป็นของคุณ"
- Log PII ใน plain text = แสดงว่า "user ไม่สำคัญ"
- Share data กับ third party โดยไม่บอก = แสดงว่า "user ไม่มีสิทธิ์รู้"

VS

- เก็บเท่าที่จำเป็น = "ฉันเคารพ user"
- Encrypt + protect = "ฉันรับผิดชอบ"
- Transparent = "ฉันยอมรับว่า user ควบคุม data ของตัวเอง"

> **คนที่ดูแลข้อมูลคนอื่นได้ดี — มี trustworthy character ที่ใช้ได้ทุกเรื่องในชีวิต**

ในชีวิต:

- คนที่เก็บความลับเก่ง = friends ไว้ใจ
- คนที่ไม่นินทา = colleagues respect
- คนที่ไม่เปิดเผยเรื่องคนอื่นใน social media = คนรอบตัวสบายใจ

ทุกคนเขียน code อะไร — สะท้อนว่า **คุณเคารพคนอื่นแค่ไหน**

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไป Part 8 — **Zero Trust Architecture** — แนวคิดของ security ที่ "ไม่เชื่อใครแม้แต่คนใน network เดียวกัน"

---

## อภิธานศัพท์ (Glossary)

- **PDPA:** พระราชบัญญัติคุ้มครองข้อมูลส่วนบุคคล (Thailand)
- **GDPR:** General Data Protection Regulation (EU)
- **PDPC / สคส.:** คณะกรรมการคุ้มครองข้อมูลส่วนบุคคล
- **Personal Data:** ข้อมูลที่ระบุตัวตน (โดยตรงหรืออ้อม)
- **Sensitive Data:** ข้อมูลที่ละเอียดอ่อน (ศาสนา, สุขภาพ, ฯลฯ)
- **PII (Personally Identifiable Information):** ข้อมูลส่วนบุคคล (US term)
- **PHI (Protected Health Information):** ข้อมูลสุขภาพ (HIPAA)
- **Data Subject:** เจ้าของข้อมูล
- **Data Controller:** ผู้ควบคุมข้อมูล (ตัดสินใจ purpose + means)
- **Data Processor:** ผู้ประมวลผลข้อมูล (vendor ที่ทำให้ controller)
- **DPO (Data Protection Officer):** คนรับผิดชอบ privacy
- **DPA (Data Processing Agreement):** สัญญาระหว่าง controller-processor
- **DSAR (Data Subject Access Request):** การขอสิทธิ์ของ user
- **Privacy by Design:** หลักออกแบบให้ privacy ตั้งแต่แรก
- **Data Minimization:** เก็บเท่าที่จำเป็น
- **Purpose Limitation:** ใช้ตามวัตถุประสงค์ที่แจ้ง
- **Anonymization:** ทำให้ระบุตัวตนไม่ได้ (ย้อนกลับไม่ได้)
- **Pseudonymization:** แทนด้วย pseudonym (ย้อนกลับได้ถ้ามี key)
- **k-Anonymity:** มาตรฐาน anonymization ทางคณิตศาสตร์
- **Differential Privacy:** เทคนิคทางคณิตศาสตร์ที่ add noise เพื่อปกป้อง
- **Right to be Forgotten:** สิทธิ์ขอให้ลบ
- **Right to Portability:** สิทธิ์ขอ data ในรูปแบบ portable
- **SCC (Standard Contractual Clauses):** EU template สำหรับ data transfer
- **BCR (Binding Corporate Rules):** internal corporate rules สำหรับ transfer
- **Cookie Consent:** การขออนุญาตใช้ cookies

---

## สรุป

1. **PDPA + GDPR** — Dev ต้องเข้าใจ ไม่ใช่แค่เรื่องของฝ่ายกฎหมาย
2. **Personal data กว้าง** — รวม IP, cookie ID, device ID
3. **Sensitive data** — ต้องการ explicit consent + extra protection
4. **Data subject rights 8 ข้อ** — ต้อง implement เป็น feature
5. **Data classification** ก่อนปกป้อง — Tier 1-4
6. **Privacy by Design** — ออกแบบให้ปลอดภัยตั้งแต่แรก
7. **Data minimization + retention** — เก็บเท่าที่จำเป็น + ลบเมื่อหมดความจำเป็น
8. **DPA กับ vendor** — ทุก third party ที่ touch PII ต้องมี
9. **Breach notification** ใน 72 ชั่วโมง
10. **โทษหนัก** — ปรับสูงสุด 5M THB + จำคุก
11. **Privacy = Respect** — ไม่ใช่แค่ compliance

ในบทถัดไป Part 8 — **Zero Trust Architecture** — security ในยุคที่ "perimeter" หายไป

— Claude Opus 4.6
