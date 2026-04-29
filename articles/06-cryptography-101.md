# บทความที่ 6: Cryptography 101 — เข้าใจ Encryption, Hashing, และ Digital Signatures

**CyberSecurity Awareness Series — Part 3: Cryptography พื้นฐาน**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องที่ทุกคน "เคยเห็น" แต่ไม่เคยเข้าใจ

ทุกครั้งที่คุณเปิดเว็บไซต์แล้วเห็น **🔒 ตัวล็อก** ข้างๆ URL — มันคืออะไรกันแน่ครับ?

ทุกครั้งที่คุณ commit code แล้ว GitHub แสดง **"Verified"** สีเขียวข้างๆ commit hash — มันการันตีอะไร?

ทุกครั้งที่ระบบ login เก็บรหัสผ่านของคุณแล้วบอกว่า "**hashed**" ไม่ใช่ "encrypted" — สองคำนี้ต่างกันยังไง?

ทุกครั้งที่ AWS / GCP บอกว่าข้อมูลคุณถูก **"encrypted at rest with AES-256"** — มันแปลว่าอะไร และเชื่อถือได้แค่ไหน?

ทุกครั้งที่คุณซื้อของออนไลน์ มี **digital signature** ของธนาคารยืนยันว่าธุรกรรมมาจากตัวคุณจริง — มันรู้ได้ยังไง?

ถ้าคุณตอบคำถามเหล่านี้ได้ทุกข้อ — คุณคือ **dev สาย senior** ที่หาได้ยากครับ ส่วนคนที่ตอบไม่ได้ครบ ไม่ต้องอาย เพราะคนส่วนใหญ่ในวงการ tech ก็ตอบไม่ได้เหมือนกัน

ปัญหาคือ — **เราใช้ cryptography ทุกวัน โดยไม่เข้าใจมันเลย** ครับ และเมื่อเราไม่เข้าใจ เราก็พลาด:

- Hash รหัสผ่านด้วย MD5 (ที่ใช้ไม่ได้ตั้งแต่ปี 2004)
- เก็บ API key ไว้ใน Git repo (โดยคิดว่า private repo = ปลอดภัย)
- ใช้ `Math.random()` สร้าง token (แทนที่จะใช้ `crypto.randomBytes`)
- เข้ารหัสข้อมูลแล้วลืม IV (initialization vector)
- Implement RSA เอง เพราะคิดว่า "ก็แค่คณิตศาสตร์"

ทุกข้อข้างต้นคือ vulnerabilities ที่เกิดขึ้นจริงในบริษัทใหญ่ๆ — ไม่ใช่เพราะ dev โง่ แต่เพราะ **ไม่มีใครสอนพื้นฐาน cryptography ในแบบที่เข้าใจง่าย**

บทความนี้จะเปลี่ยนเรื่องนั้นครับ ผมจะอธิบายให้คุณ "เข้าใจพอที่จะใช้ถูก" — ไม่ใช่ "เข้าใจพอจะ implement เอง" (ซึ่งต่างกันมาก และคุณ **ไม่ควร** implement เอง)

---

## ทำไม Dev ต้องเข้าใจ Cryptography (ไม่ต้องเป็นนักคณิตศาสตร์)

มีคนเข้าใจผิดเรื่อง cryptography 2 แบบ:

**แบบที่ 1:** "ฉันไม่ใช่นักคณิตศาสตร์ ทำงานสาย backend / frontend ธรรมดา ไม่ต้องรู้เรื่องนี้"

**แบบที่ 2:** "ฉันรู้พอแล้ว เคยอ่าน Bruce Schneier มาบ้าง — ฉันลอง implement crypto algorithm เองได้เลย"

ทั้งสองแบบอันตรายครับ:

### แบบที่ 1 อันตรายเพราะ — Crypto อยู่ในทุกระบบ

- Login system → password hashing
- API call → TLS/HTTPS
- JWT token → digital signature
- File storage → encryption at rest
- Inter-service communication → mutual TLS
- Mobile app ↔ backend → certificate pinning
- Code deployment → signed packages
- Secret management → encrypted env variables

**คุณอาจจะไม่ได้ "เขียน crypto" เอง — แต่คุณกำลัง "ใช้" มันทุกวัน** และถ้าใช้ผิด ระบบทั้งระบบล่ม

### แบบที่ 2 อันตรายเพราะ — Crypto ที่ implement เองมักจะแย่

> **กฎเหล็กของ cryptography: "Never roll your own crypto"** — Bruce Schneier

มี algorithm ที่นักคณิตศาสตร์อัจฉริยะใช้เวลาหลายปี และมีการ peer review จากนักวิจัยทั่วโลก ก่อนที่จะถูกพิสูจน์ว่า "อาจจะปลอดภัย" — คุณคิดว่าคุณจะเขียนได้ดีกว่าคนพวกนั้น ในเวลาที่น้อยกว่าเขา ในระบบที่ไม่มีใคร review เลยเหรอครับ?

ข้อผิดพลาดที่เกิดบ่อยเมื่อ implement crypto เอง:

1. **Side-channel attack** — ใช้เวลา compare string ต่างกันตามตำแหน่งของตัวที่ผิด → ผู้โจมตีดู timing แล้ว infer key ได้
2. **Weak randomness** — ใช้ `random()` ที่ predict ได้ แทนที่จะใช้ `secureRandom()`
3. **No padding / wrong padding** — encrypt ข้อมูลที่ไม่ตรง block size ทำให้รั่ว
4. **No authentication** — encrypt ได้ แต่ไม่รู้ว่าข้อมูลถูกแก้ไขหรือไม่ (ต้องใช้ authenticated encryption เช่น AES-GCM)
5. **IV reuse** — ใช้ initialization vector ซ้ำ → บาง algorithm รั่วทันที

> **สิ่งที่ต้องรู้คือ "เมื่อไหร่ใช้อะไร" — ไม่ใช่ "implement ยังไง"**

---

## Encryption — สอง flavor ที่ต้องแยกให้ออก

**Encryption** คือการแปลงข้อมูลที่อ่านออก (plaintext) เป็นข้อมูลที่อ่านไม่ออก (ciphertext) โดยที่สามารถ "ย้อนกลับ" ได้ถ้ามี **key** ที่ถูกต้อง

มี 2 flavor ใหญ่ๆ:

### 1. Symmetric Encryption — กุญแจดอกเดียว

**Symmetric encryption** ใช้ **กุญแจเดียวกัน** ทั้งล็อกและปลดล็อก

```
plaintext + key  →  [encrypt]  →  ciphertext
ciphertext + key →  [decrypt]  →  plaintext
```

**Algorithm ที่ควรรู้:**

- **AES** (Advanced Encryption Standard) — มาตรฐานตั้งแต่ปี 2001 ใช้ทุกที่ในโลก
  - **AES-128** / **AES-256** — ตัวเลขคือขนาด key (bits)
  - **AES-GCM** — โหมดที่แนะนำ มี authentication ในตัว (กันการแก้ไข ciphertext)
  - **AES-CBC** — โหมดเก่า ต้องใช้คู่กับ HMAC ถึงจะปลอดภัย
- **ChaCha20-Poly1305** — modern alternative ของ AES-GCM ออกแบบมาให้เร็วบน device ที่ไม่มี hardware AES

**ตัวอย่างการใช้งานจริง:**

- **Database encryption at rest** — encrypt ทั้ง disk
- **File encryption** — เช่น VeraCrypt, LUKS
- **TLS session** — หลัง handshake แล้ว ใช้ symmetric key ในการ encrypt traffic ระหว่าง browser ↔ server
- **Password manager vault** — encrypt ด้วย key ที่ derive จาก master password

**ข้อดี:** เร็วมาก — encrypt ข้อมูลขนาด GB ได้ในไม่กี่วินาที

**ข้อเสียที่สำคัญ:** ต้องแชร์ key กันระหว่างผู้ส่งและผู้รับ — แล้วจะแชร์ยังไงให้ปลอดภัย? นี่คือปัญหาที่ asymmetric encryption แก้ได้

### 2. Asymmetric Encryption — กุญแจคู่

**Asymmetric encryption** หรือ **Public Key Cryptography** ใช้กุญแจ **คู่** — public key (เปิดเผยได้) และ private key (เก็บเป็นความลับ)

```
plaintext + public_key  →  [encrypt]  →  ciphertext
ciphertext + private_key →  [decrypt]  →  plaintext
```

จุดที่ "เวทย์มนต์" คือ — **อะไรที่ encrypt ด้วย public key จะ decrypt ได้ด้วย private key เท่านั้น** (และทางกลับกันก็ได้สำหรับการ sign)

**Algorithm ที่ควรรู้:**

- **RSA** — ตัวคลาสสิก ใช้กันมานานตั้งแต่ปี 1977
  - **RSA-2048** / **RSA-4096** — ขนาด key (bits) ปัจจุบันแนะนำขั้นต่ำ 2048
- **ECC** (Elliptic Curve Cryptography) — modern alternative ของ RSA
  - **ECDSA, EdDSA** — สำหรับ digital signatures
  - **ECDH** — สำหรับ key exchange
  - **Ed25519, secp256r1** — curves ที่นิยม
  - ข้อดี: key เล็กกว่า RSA แต่ปลอดภัยพอๆ กัน (256-bit ECC ≈ 3072-bit RSA)

**ตัวอย่างการใช้งานจริง:**

- **TLS handshake** — เปลี่ยน symmetric key ระหว่าง browser ↔ server แบบปลอดภัย
- **SSH keys** — ที่คุณ `ssh-keygen` มาใช้ login GitHub
- **PGP/GPG** — encrypt email, sign git commit
- **Cryptocurrency** — Bitcoin, Ethereum ใช้ ECC ทั้งหมด
- **Passkey / FIDO2** — ที่เราพูดในบทที่ 5

**ข้อดี:** แก้ปัญหา key distribution ได้ — ใครก็ encrypt มาให้ผมได้ ถ้ามี public key ของผม โดยที่ไม่ต้องแชร์อะไรลับๆ มาก่อน

**ข้อเสีย:** ช้ากว่า symmetric หลายเท่า — ไม่เหมาะ encrypt ข้อมูลขนาดใหญ่

### Hybrid: ใช้ทั้งคู่ — ที่ TLS ทำอยู่

ในระบบจริง เราใช้ทั้งคู่ผสมกัน เพื่อเอาข้อดีของแต่ละแบบ:

1. **ใช้ asymmetric** เพื่อแลกเปลี่ยน symmetric key (เร็วพอ เพราะ key มันเล็ก)
2. **ใช้ symmetric** ในการ encrypt ข้อมูลที่ส่งจริง (เร็ว เพราะข้อมูลใหญ่)

นี่คือสิ่งที่ TLS ทำเวลาคุณเปิด `https://...` ครับ — เราจะเข้าเรื่องนี้กันต่อในส่วนของ TLS

---

## Hashing vs Encryption — ความสับสนที่ต้องแก้ครั้งเดียวจบ

นี่คือเรื่องที่ทำให้ dev มือใหม่ (และมือเก๋าบางคน) สับสนบ่อยมากครับ

### สรุปความต่างใน 1 ประโยค

> **Encryption ย้อนกลับได้ (ถ้ามี key)** — **Hashing ย้อนกลับไม่ได้ (ตลอดไป)**

### ตารางเปรียบเทียบ

| ประเด็น | Encryption | Hashing |
|---|---|---|
| **ทิศทาง** | สองทาง (encrypt ↔ decrypt) | ทางเดียว (forward only) |
| **Key** | ต้องมี key | ไม่ใช้ key (แค่ algorithm) |
| **Output size** | แปรตาม input | ขนาดคงที่ (เช่น 256-bit เสมอ) |
| **Use case** | ข้อมูลที่ต้องอ่านกลับ | ข้อมูลที่ไม่ต้องการอ่านกลับ แค่เปรียบเทียบ |
| **Algorithm** | AES, RSA, ChaCha20 | SHA-256, SHA-3, bcrypt, Argon2 |

### Hashing ทำงานยังไง

**Hash function** รับ input ขนาดเท่าไหร่ก็ได้ → return string ขนาดคงที่ (เรียกว่า **digest** หรือ **hash**)

```
SHA-256("hello")
  → 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824

SHA-256("Hello")  // เปลี่ยน h → H
  → 185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969

SHA-256("a very long input that has megabytes of data...")
  → ขนาดเท่าเดิม 64 hex characters (256 bits)
```

คุณสมบัติของ hash function ที่ดี:

1. **Deterministic** — input เดียวกันได้ output เดียวกันเสมอ
2. **One-way** — รู้ output แล้วย้อนกลับไปหา input ไม่ได้ (ในเวลาที่ทำได้จริง)
3. **Avalanche effect** — เปลี่ยน input นิดเดียว output เปลี่ยนหมด
4. **Collision-resistant** — หา input 2 ตัวที่ได้ output เดียวกัน (ในทางปฏิบัติ) ทำไม่ได้

### Hash ใช้ทำอะไร

**1. ตรวจสอบความถูกต้องของไฟล์**

```bash
# ตอน download
sha256sum ubuntu.iso
# c2c4f1d... (ต้องตรงกับที่เว็บประกาศไว้)
```

ถ้าไฟล์ถูกแก้ไขแม้แต่ bit เดียว hash จะเปลี่ยนหมด — รู้ทันที

**2. เก็บรหัสผ่าน**

แทนที่จะเก็บ "myPassword123" ตรงๆ ใน DB เราเก็บ hash ของมันแทน:

```
DB:
  username: "tanin"
  password_hash: "$2b$12$N9qo8uLOickgx2ZMRZoMye..."

ตอน login:
  user ป้อน password "myPassword123"
  → hash ด้วย bcrypt → ได้ "$2b$12$N9qo8uLOickgx2ZMRZoMye..."
  → เปรียบเทียบกับ hash ใน DB → match → login ได้
```

ถ้า DB หลุด — ผู้โจมตีเห็นแต่ hash ไม่เห็นรหัสผ่านจริง (ถ้าใช้ algorithm ที่ดี)

**3. Digital Signatures (ภายใน)**

ก่อน sign ข้อมูล เรา hash ก่อน แล้วค่อย sign hash (เพราะ hash ขนาดคงที่ ทำให้ sign เร็ว) — เราจะลงรายละเอียดในส่วนถัดไป

**4. Blockchain**

แต่ละ block ใน Bitcoin/Ethereum ผูกด้วย hash ของ block ก่อนหน้า — เลย "tamper-evident" ถ้าใครพยายามแก้ block เก่า hash ทั้งสายก็ต้องคำนวณใหม่หมด

### Hash function ที่ควรใช้

**สำหรับ general purpose hashing (file integrity, blockchain ฯลฯ):**

- ✅ **SHA-256, SHA-3** — เร็ว ปลอดภัย ใช้ได้ทั่วไป
- ✅ **BLAKE2 / BLAKE3** — เร็วกว่า SHA แต่ปลอดภัยพอกัน

**สำหรับ password hashing — ต้องใช้คนละแบบ:**

- ✅ **Argon2id** — ตัวที่แนะนำในปี 2026 (ชนะ Password Hashing Competition ปี 2015)
- ✅ **bcrypt** — เก่ากว่า แต่ใช้ได้ ปลอดภัย
- ✅ **scrypt** — ทางเลือกอีกตัว
- ❌ **PBKDF2** — เก่า ใช้ได้แต่ไม่แนะนำสำหรับ password ใหม่

### ทำไม password ต้องใช้ algorithm พิเศษ?

จุดประสงค์ของ password hashing **ต่างกัน** จาก general hashing — เราต้องการให้มัน **ช้าลง** จงใจ

**ปัญหาของ SHA-256 สำหรับ password:**

- SHA-256 เร็วมาก — GPU สมัยใหม่คำนวณ SHA-256 ได้หลายพันล้านครั้งต่อวินาที
- ผู้โจมตีที่ได้ DB หลุด ก็ใช้ GPU ลอง brute force ทุก password ที่เป็นไปได้
- รหัสผ่านยาว 8 ตัวที่เป็น lowercase + number — แค่ไม่กี่นาทีก็แตก

**สิ่งที่ bcrypt / Argon2 ทำคือ:**

1. **ช้า by design** — แม้บน GPU ก็ใช้เวลาคำนวณนานกว่า (มี cost factor ปรับได้)
2. **Memory-hard** (Argon2) — ใช้ RAM เยอะ ทำให้ GPU/ASIC ไม่ได้เปรียบ
3. **Salt อัตโนมัติ** — รหัสผ่านเดียวกันของ user 2 คน ได้ hash ต่างกัน

### Salt — สิ่งที่ทำให้ password hashing ปลอดภัย

ถ้าเราแค่ `SHA-256("password123")` ตรงๆ — รหัสผ่านเดียวกันของทุกคนจะได้ hash เดียวกัน

ผู้โจมตีจึงทำตารางที่เรียกว่า **rainbow table** — pre-compute hash ของ password ที่นิยมไว้ล่วงหน้า:

```
"password"  → 5e88489...
"123456"    → 8d969ee...
"qwerty"    → 65e84be...
... ล้านล้าน entries
```

เห็น hash ใน DB → look up rainbow table → เจอรหัสผ่านทันที

**Salt** คือสตริงสุ่มที่ append (หรือ prepend) เข้ากับ password ก่อน hash:

```
hash("password123" + "x9k2lm83p1") → ABC123...
hash("password123" + "8h2nf91kx2") → DEF456...
```

User คนละคน → salt ต่างกัน → hash ต่างกัน — แม้รหัสผ่านจริงเป็นตัวเดียวกัน Rainbow table ใช้ไม่ได้

bcrypt และ Argon2 generate salt อัตโนมัติและฝังไว้ใน hash output — ดังนั้นคุณแค่เรียกใช้ ไม่ต้องจัดการเอง

```
bcrypt output: $2b$12$N9qo8uLOickgx2ZMRZoMye...
                ^^   ^^^^^^^^^^^^^^^^^^^^^^^
                cost      salt + hash
```

### ❌ Algorithm ที่ "ห้ามใช้แล้ว"

- **MD5** — แตกตั้งแต่ปี 2004 ห้ามใช้
- **SHA-1** — แตก collision ในปี 2017 ห้ามใช้สำหรับ security
- **SHA-256 ตรงๆ สำหรับ password** — เร็วเกินไป ใช้ bcrypt/Argon2 แทน
- **DES, 3DES** — เก่า ห้ามใช้ ใช้ AES แทน
- **RC4** — ห้ามใช้

---

## Digital Signatures — พิสูจน์ว่าใครเป็นคนส่ง และข้อมูลไม่ถูกแก้ไข

**Digital signature** ใช้เพื่อตอบคำถาม 2 ข้อ:

1. **Authentication** — ข้อมูลนี้มาจาก "คนนั้น" จริงไหม?
2. **Integrity** — ข้อมูลนี้ถูกแก้ไขระหว่างทางหรือเปล่า?

### หลักการ — กลับด้านของ Asymmetric Encryption

จำได้ไหมครับว่า asymmetric encryption ใช้ public key encrypt → private key decrypt?

Digital signature ใช้คู่กลับด้าน:

```
Sign:    data + private_key  →  signature
Verify:  data + signature + public_key  →  valid? (true/false)
```

ขั้นตอนจริงๆ:

```
1. Hash ข้อมูลก่อน (ให้ขนาดคงที่)
   hash = SHA-256(data)

2. Sign hash นั้นด้วย private key
   signature = sign(hash, private_key)

3. ส่ง data + signature ไปด้วยกัน

4. ผู้รับ verify:
   - Hash data ที่ได้รับ → expected_hash
   - Decrypt signature ด้วย public key → received_hash
   - ถ้า expected_hash == received_hash → valid
```

### ทำไมเชื่อได้ว่า "มาจากคนนั้นจริง"?

เพราะ **มีแค่คนคนนั้น** ที่มี private key → มีแค่เขาที่ sign ได้ → ถ้า verify ผ่านด้วย public key ของเขา = เขาเป็นคนสร้าง signature นี้แน่ๆ

(ถ้าใครได้ private key ของคุณไป — ก็ปลอมเป็นคุณได้ — นี่คือเหตุผลว่าทำไม private key ห้ามรั่วเด็ดขาด)

### ตัวอย่างที่คุณใช้ทุกวัน

**1. Git commit signing**

```bash
git config --global commit.gpgsign true
git config --global user.signingkey YOUR_KEY_ID
git commit -m "fix bug"
# Commit นี้ถูก sign ด้วย GPG key ของคุณ
```

GitHub แสดง **"Verified"** เขียวๆ เมื่อ public key ของคุณถูก register ไว้ใน account แล้ว — ทำให้รู้ว่า commit นี้คุณเขียนจริง ไม่ใช่ใครปลอมเป็นคุณ

**2. Software updates**

ทุกครั้งที่ Windows / macOS / iOS update — package update ถูก sign ด้วย private key ของ Microsoft / Apple ระบบ verify signature ก่อนติดตั้ง ถ้า signature ผิด = อาจมีคนแก้ไขระหว่างทาง = ไม่ติดตั้ง

**3. JWT (JSON Web Token)**

```
header.payload.signature
```

ส่วน signature คือ JWT ที่ sign ด้วย key ของ server — ทำให้ client แก้ payload ไม่ได้ (จะแก้ก็ได้ แต่ signature ไม่ผ่าน verify)

**4. TLS Certificates**

เราจะเข้าเรื่องนี้ในส่วนถัดไป — แต่หัวใจคือ digital signature เหมือนกัน

**5. Cryptocurrency Transactions**

ทุก transaction ใน Bitcoin / Ethereum ถูก sign ด้วย private key ของเจ้าของ wallet — เพื่อพิสูจน์ว่าเจ้าของยินยอม

### Algorithm สำหรับ Digital Signature

- ✅ **ECDSA** (P-256, P-384) — มาตรฐานปัจจุบัน
- ✅ **EdDSA / Ed25519** — modern, ไม่มี edge cases เหมือน ECDSA
- ✅ **RSA-PSS** — ถ้าใช้ RSA ใช้ตัวนี้ ไม่ใช่ RSA-PKCS1
- ⚠️ **RSA-PKCS1 v1.5** — เก่า มี edge cases ควรเปลี่ยนเป็น RSA-PSS

---

## TLS/SSL — เกิดอะไรขึ้นเมื่อเปิด HTTPS

ทุกครั้งที่คุณเปิด `https://google.com` มีเรื่องเกิดขึ้นมากมายในเสี้ยววินาทีครับ ผมจะอธิบายเป็นขั้นตอน

### TCP Handshake (ก่อนหน้าทุกอย่าง)

ก่อน TLS เริ่ม browser ต้อง connect TCP ก่อน — 3-way handshake ปกติ (SYN, SYN-ACK, ACK)

### TLS 1.3 Handshake (ปัจจุบัน — เร็วกว่า TLS 1.2)

```
[Browser]                           [Server]
   |                                    |
   |--- ClientHello ------------------->|
   |    (รายการ cipher ที่รองรับ +     |
   |     ECDHE public key ของ client)  |
   |                                    |
   |<-- ServerHello --------------------|
   |    (เลือก cipher +                |
   |     ECDHE public key ของ server + |
   |     Certificate +                  |
   |     CertificateVerify +            |
   |     Finished)                      |
   |                                    |
   |--- Finished --------------------- >|
   |                                    |
   |==== Encrypted application data ===|
```

**สิ่งที่เกิดขึ้น:**

1. **Browser ส่ง "ClientHello"** — บอก server ว่า "ฉันรองรับ cipher อะไรบ้าง" + ส่ง ephemeral public key
2. **Server ส่ง "ServerHello"** — เลือก cipher + ส่ง public key ของ session + **ส่ง certificate ของ server** + sign ทุกอย่างด้วย private key ของ server
3. **Browser verify certificate** — เช็คว่า certificate มาจาก CA ที่เชื่อถือได้, signature ถูกต้อง, domain match
4. **ทั้งสองคำนวณ shared secret** ด้วย ECDHE (Elliptic Curve Diffie-Hellman Ephemeral) — ได้ symmetric key ที่ทั้งสองรู้ แต่ใครดักฟังก็ไม่ได้
5. **เริ่ม encrypt traffic ด้วย symmetric key** — ทุก byte หลังจากนั้น encrypt ด้วย AES-GCM หรือ ChaCha20-Poly1305

> **TL;DR:** TLS = ใช้ asymmetric ในการแลก symmetric key + verify ตัวตน server แล้วใช้ symmetric encrypt ทุกอย่างจริงๆ

### Certificate และ Certificate Authority (CA)

**Certificate** คือเอกสารดิจิทัลที่บอกว่า "public key นี้เป็นของ domain นี้" — sign ด้วย CA

**Certificate Authority (CA)** คือองค์กรที่ browser/OS เชื่อถือไว้ก่อน เช่น:

- Let's Encrypt (ฟรี ใช้กันแพร่หลายที่สุด)
- DigiCert
- Sectigo
- GoDaddy

```
google.com Certificate
  ├─ Subject: google.com
  ├─ Public Key: 04:9d:4f:...
  ├─ Issuer: GTS CA 1C3
  └─ Signature (sign โดย private key ของ GTS CA 1C3)

GTS CA 1C3 Certificate (intermediate)
  ├─ Subject: GTS CA 1C3
  ├─ Public Key: ...
  ├─ Issuer: GTS Root R1
  └─ Signature (sign โดย private key ของ GTS Root R1)

GTS Root R1 (root)
  ├─ Subject: GTS Root R1
  ├─ Public Key: ...
  └─ ⭐ อยู่ใน "trust store" ของ OS / Browser
      (Mac, Windows, Chrome, Firefox มาพร้อมรายการ root CA ที่เชื่อถือ)
```

นี่เรียกว่า **certificate chain** — สาย verify ที่ไล่จาก leaf (domain) → intermediate → root

Browser **verify จาก leaf ขึ้นไปเรื่อยๆ** จน root ต้องอยู่ใน trust store

### ทำไม Self-Signed Certificate ถึงเตือน?

**Self-signed certificate** = sign ด้วย private key ตัวเอง ไม่มี CA รับรอง

```
my-server.local Certificate
  ├─ Subject: my-server.local
  ├─ Issuer: my-server.local  ← issuer เป็นตัวเอง
  └─ Signature (sign โดย private key ของตัวเอง)
```

ปัญหา: **ใครๆ ก็สร้างได้** — ผู้โจมตีก็สร้าง self-signed cert ของ google.com ได้ในเสี้ยววินาที

ดังนั้น browser ไม่เชื่อ self-signed cert โดย default — เพราะไม่มีอะไรพิสูจน์ได้ว่ามันเป็นของจริง

**ใช้ self-signed cert เมื่อไหร่?**

- ✅ Development environment local
- ✅ Internal services ที่ deploy custom CA
- ❌ Production website ที่ user เห็น

### HTTPS ป้องกันอะไร — ไม่ป้องกันอะไร

**ป้องกัน:**

- ✅ ดักฟังข้อมูลระหว่างทาง (eavesdropping)
- ✅ แก้ไขข้อมูลระหว่างทาง (tampering)
- ✅ ปลอมตัวเป็น server (impersonation) — ต้องผ่าน CA verification

**ไม่ป้องกัน:**

- ❌ ถ้า server เองโดนแฮก
- ❌ ถ้า client เอง (cookie โดนขโมย, malware ในเครื่อง)
- ❌ Phishing ที่ใช้ domain คล้ายๆ (เช่น `g00gle.com` — มี cert จริง แต่ไม่ใช่ Google)
- ❌ การ leak ข้อมูลผ่าน metadata (ดูได้ว่าเข้าเว็บไหน เมื่อไหร่ ขนาดเท่าไหร่)

---

## Encryption at Rest vs In Transit

นี่คือสองคำที่คุณจะได้ยินบ่อยในการ design ระบบ:

### Encryption in Transit — ปกป้องตอนข้อมูลเดินทาง

หมายถึง encrypt **ระหว่างที่ข้อมูลถูกส่ง** จากที่หนึ่งไปอีกที่หนึ่ง

**ตัวอย่าง:**

- HTTPS (browser → web server)
- SSH (terminal → remote server)
- VPN (device → corporate network)
- mTLS (microservice → microservice)
- TLS database connection (app → database)

### Encryption at Rest — ปกป้องตอนข้อมูลถูกเก็บ

หมายถึง encrypt ข้อมูล **ที่ถูกเก็บไว้** ใน disk, database, backup ฯลฯ

**ตัวอย่าง:**

- AWS S3 server-side encryption
- AWS RDS encryption
- BitLocker (Windows), FileVault (Mac), LUKS (Linux)
- Database TDE (Transparent Data Encryption)
- Backup encryption

### ทำไมต้องมีทั้งสอง?

ลองนึกภาพ:

- Encrypt **in transit อย่างเดียว** → ข้อมูลปลอดภัยตอนเดินทาง แต่ถ้า disk ของ server โดนขโมย = หมด
- Encrypt **at rest อย่างเดียว** → ข้อมูลปลอดภัยใน disk แต่ถ้าใครดัก traffic = หมด

**คุณต้องป้องกันทั้งสอง** — ไม่ใช่เลือกอย่างใดอย่างหนึ่ง

### "End-to-End Encryption" ต่างจาก "in transit" ยังไง?

**E2E (End-to-End) Encryption** = encrypt ที่ฝั่ง sender, decrypt ที่ฝั่ง receiver **เท่านั้น** — server ระหว่างทางอ่านไม่ได้แม้แต่ตัวเอง

ตัวอย่าง E2E:
- **Signal** — server ของ Signal เห็นแต่ ciphertext
- **iMessage** — Apple อ่านข้อความคุณไม่ได้
- **WhatsApp** — Meta อ่านข้อความไม่ได้ (theoretically)

ปกติ "in transit" หมายถึง encrypt ระหว่าง client ↔ server เท่านั้น — server เห็น plaintext ได้ ตัวอย่างเช่น Gmail — Google อ่าน email คุณได้ (และ scan เพื่อ AI/Ads)

E2E ต้องใช้ asymmetric crypto + key management ที่ซับซ้อนกว่า — ปกติแอป chat ที่จริงจังเรื่อง privacy ถึงจะมี

---

## Key Management — เรื่องที่ทำพลาดบ่อยที่สุด

> **The hardest part of cryptography is not the math — it's key management.**

**Algorithm** ทุกตัวอันแข็งแกร่ง พังเมื่อ key:

- Hardcode ใน source code
- เก็บใน Git repo (แม้ private)
- ส่งทาง email / Slack / chat
- ใช้ key เดียวกันทุกที่ทุกระบบ
- ไม่เคย rotate

### หลักการ Key Management

**1. Key ต้อง random จริงๆ**

ใช้ cryptographically secure random:

```javascript
// Node.js
const { randomBytes } = require('crypto');
const key = randomBytes(32);  // 256-bit AES key
```

```python
# Python
import secrets
key = secrets.token_bytes(32)
```

❌ **ห้ามใช้** `Math.random()`, `random.random()`, `rand()` — predictable!

**2. Key ต้องเก็บแยกจาก data**

ห้ามเก็บ key ใน:
- Source code → Git repo รั่ว = key รั่ว
- Config file ใน repo เดียวกัน → เหมือนกัน
- Environment variable ที่ commit ลง `.env` file → เหมือนกัน

ที่เก็บที่ถูกต้อง:
- ✅ **Cloud KMS** (AWS KMS, GCP Cloud KMS, Azure Key Vault)
- ✅ **HashiCorp Vault**
- ✅ **Hardware Security Module (HSM)** — สำหรับ key สำคัญสุดๆ
- ✅ **Sealed secrets / SOPS** — สำหรับ infra-as-code

**3. Key Rotation**

Key มีอายุการใช้งาน — ใช้นานๆ ความเสี่ยงสะสม:

- Symmetric key สำหรับ data → rotate ปีละครั้ง
- API key → rotate ทุก 90 วัน
- TLS certificate → ปกติ 90 วัน (Let's Encrypt) ถึง 1 ปี
- Signing key สำคัญ → 5-10 ปี (เป็นเรื่องใหญ่ทุกครั้งที่ rotate)

**4. Key Revocation**

เมื่อ key รั่ว / สงสัยรั่ว / พนักงาน access ออกจากบริษัท — ต้อง **revoke** ได้ทันที โดยไม่กระทบระบบทั้งหมด

**5. Principle of Least Privilege**

แต่ละ service / user ได้แค่ key ที่จำเป็น — ไม่ใช่ master key ที่เปิดได้ทุกอย่าง

---

## Common Mistakes — สิ่งที่ Dev ทำพลาดเป็นประจำ

### Mistake 1: ใช้ MD5 / SHA-1 สำหรับ password

```python
# ❌ WRONG
import hashlib
hashed = hashlib.md5(password.encode()).hexdigest()
```

**MD5 / SHA-1 ห้ามใช้ทั้งคู่ทั้งกับ password และ general security** — แตกได้เป็นล้านเท่า

```python
# ✅ CORRECT
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
```

### Mistake 2: Implement Crypto เอง

```javascript
// ❌ WRONG — ใครเขียนแบบนี้ ขอให้ตื่น
function myEncrypt(data, key) {
  let result = '';
  for (let i = 0; i < data.length; i++) {
    result += String.fromCharCode(data.charCodeAt(i) ^ key.charCodeAt(i % key.length));
  }
  return result;
}
```

นี่คือ XOR cipher — แตกใน 5 นาที

```javascript
// ✅ CORRECT
const { createCipheriv, randomBytes } = require('crypto');
const iv = randomBytes(12);
const cipher = createCipheriv('aes-256-gcm', key, iv);
const encrypted = Buffer.concat([cipher.update(data), cipher.final()]);
const authTag = cipher.getAuthTag();
```

### Mistake 3: ลืม Salt ใน Password Hashing

```python
# ❌ WRONG
hashed = hashlib.sha256(password.encode()).hexdigest()  # ไม่มี salt!
```

```python
# ✅ CORRECT
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt())  # bcrypt salt อัตโนมัติ
```

### Mistake 4: ใช้ ECB Mode

```javascript
// ❌ WRONG — ECB mode รั่ว pattern
const cipher = createCipheriv('aes-256-ecb', key, null);
```

ECB mode encrypt block ละ block แยกกัน — block ที่เหมือนกันได้ ciphertext เหมือนกัน → รั่ว pattern

ตัวอย่างคลาสสิกคือ "ECB Penguin" — encrypt รูปนกเพนกวินด้วย ECB แล้วยังมองออกว่าเป็นเพนกวิน

```javascript
// ✅ CORRECT — ใช้ GCM (มี authentication ด้วย)
const cipher = createCipheriv('aes-256-gcm', key, iv);
```

### Mistake 5: IV Reuse

```javascript
// ❌ WRONG — IV เดียวกันทุกครั้ง
const iv = Buffer.alloc(12, 0);  // ทั้งหมดเป็น 0
```

ใช้ IV ซ้ำกับ key เดียวกัน → ใน AES-GCM = catastrophic (เห็น plaintext ได้)

```javascript
// ✅ CORRECT
const iv = randomBytes(12);  // random ทุกครั้ง
```

### Mistake 6: เก็บ Key ใน Source Code

```python
# ❌ WRONG
SECRET_KEY = "<your-stripe-secret-key>"  # hardcoded — ห้ามทำแบบนี้!
```

```python
# ✅ CORRECT
import os
SECRET_KEY = os.environ['STRIPE_SECRET_KEY']  # จาก env, ไม่ commit
# หรือดีกว่า: ดึงจาก secret manager
```

### Mistake 7: String Comparison แบบไม่ Constant-Time

```python
# ❌ WRONG — vulnerable to timing attack
if user_token == expected_token:
    grant_access()
```

Python `==` หยุด compare ทันทีที่เจอ char ที่ไม่ตรง — ผู้โจมตีวัด timing แล้วกู้ token ทีละตัวอักษร

```python
# ✅ CORRECT
import hmac
if hmac.compare_digest(user_token, expected_token):
    grant_access()
```

### Mistake 8: ไม่ verify TLS certificate

```python
# ❌ WRONG — ทำให้ HTTPS เหมือน HTTP
import requests
requests.get("https://api.example.com", verify=False)
```

```python
# ✅ CORRECT — verify by default
requests.get("https://api.example.com")  # verify=True default
```

---

## Quick Reference — ใช้อะไรเมื่อไหร่

### "ฉันต้อง..."

| ความต้องการ | ใช้ |
|---|---|
| เก็บรหัสผ่าน user | **bcrypt** หรือ **Argon2id** |
| ตรวจสอบว่าไฟล์ไม่ถูกแก้ไข | **SHA-256** |
| Encrypt ข้อมูลก่อนเก็บ DB | **AES-256-GCM** |
| Encrypt ข้อมูลส่งระหว่าง service | **TLS** (mTLS ถ้า zero-trust) |
| Sign request เพื่อพิสูจน์ตัวตน | **HMAC-SHA256** หรือ **ECDSA** |
| Sign software / commit | **GPG / Sigstore / EdDSA** |
| Generate random token / API key | **`crypto.randomBytes()`** หรือ **`secrets.token_*()`** |
| Encrypt file สำหรับส่งให้คนอื่น | **age** หรือ **GPG** |
| Generate JWT | **HS256** (symmetric) หรือ **RS256/ES256** (asymmetric) |
| TLS certificate ฟรีสำหรับ public website | **Let's Encrypt** |
| Manage secret ในระบบ | **AWS KMS / Vault / Cloud secret manager** |

### "ฉันห้ามใช้..."

- ❌ MD5 ทุกกรณี
- ❌ SHA-1 สำหรับ security (file checksum ใช้ได้ แต่อย่าใช้กัน collision)
- ❌ DES, 3DES, RC4
- ❌ ECB mode
- ❌ Self-rolled crypto
- ❌ `Math.random()` สำหรับ security purpose
- ❌ Hardcoded keys/secrets
- ❌ HTTP สำหรับข้อมูลที่ใหญ่กว่า "อะไรไม่สำคัญ"

---

## บทเรียนชีวิตจากบทความนี้

> **คุณไม่จำเป็นต้องรู้ว่าเครื่องยนต์ทำงานยังไงทุกชิ้นส่วน — แต่ต้องรู้ว่าเมื่อไหร่ควรเติมน้ำมัน เมื่อไหร่ควรเปลี่ยนยาง**

ลองคิดเรื่องการขับรถดูครับ:

- คุณไม่ต้องรู้วิธีคำนวณ thermodynamics ของ combustion engine — แต่คุณต้องรู้ว่าเข็มน้ำมันใกล้แดงแล้วต้องไปเติม
- คุณไม่ต้องรู้สูตร friction coefficient ของยาง — แต่คุณต้องรู้ว่ายางสึกเกิน 50% แล้วต้องเปลี่ยน
- คุณไม่ต้องรู้ chemistry ของน้ำมันเครื่อง — แต่คุณต้องรู้ว่าทุก 10,000 km ต้องเปลี่ยน

**Cryptography ก็เหมือนกันครับ:**

- คุณไม่ต้องรู้สูตรคณิตศาสตร์ของ AES-256 — แต่คุณต้องรู้ว่า "ใช้ AES-256-GCM ไม่ใช่ ECB"
- คุณไม่ต้องรู้ว่า bcrypt cost factor 12 หมายถึงอะไรในระดับ CPU cycles — แต่คุณต้องรู้ว่า "ใช้ bcrypt ไม่ใช่ MD5"
- คุณไม่ต้องรู้ Galois Field theory — แต่คุณต้องรู้ว่า "TLS 1.3 ดีกว่า TLS 1.2"

**Key insight:** ในเรื่องที่ซับซ้อนเกินกว่าที่เราจะเข้าใจทั้งหมด เราใช้สิ่งที่เรียกว่า **"abstraction"** — เชื่อใจสิ่งที่ผู้เชี่ยวชาญทำมา และใช้ตามคำแนะนำของเขา

นี่คือทักษะของชีวิตที่สำคัญมาก:

- คุณไม่ต้องรู้ว่าหมอดูเลือดยังไง — แต่ต้องรู้ว่าเมื่อไหร่ควรไปตรวจสุขภาพ
- คุณไม่ต้องรู้กฎหมายทั้งฉบับ — แต่ต้องรู้ว่าเมื่อไหร่ควรปรึกษาทนาย
- คุณไม่ต้องเข้าใจ macroeconomics ระดับลึก — แต่ต้องรู้ว่ากระจายลงทุนยังไง

ในเรื่อง cryptography คำถามที่คุณต้องถามตัวเองไม่ใช่ "ฉันเข้าใจ AES ลึกแค่ไหน" แต่เป็น:

1. **ฉันใช้ algorithm ที่แนะนำในปัจจุบันหรือไม่?**
2. **ฉันใช้ library ที่ผ่านการ audit จริงไหม?**
3. **ฉันเก็บ key ไว้ในที่ที่ปลอดภัยหรือเปล่า?**
4. **ฉันมี process update เมื่อมาตรฐานเปลี่ยนไหม?**
5. **เมื่อไหร่ที่ฉันรู้สึกว่า "ฉลาดพอจะ implement crypto เอง" — ฉันต้องหยุดและถามผู้เชี่ยวชาญ**

ความถ่อมตัวต่อสิ่งที่เราไม่รู้ คือคุณสมบัติของ engineer ที่ดี — และเป็นคุณสมบัติของผู้ใหญ่ที่ดีในชีวิตเช่นกัน

---

## Action Items

### สำหรับทุกคน

- [ ] เปลี่ยน password ของบัญชีสำคัญที่สุด → ใช้ password manager + 2FA (จากบทที่ 4-5)
- [ ] ตรวจสอบว่าทุกเว็บที่คุณกรอกข้อมูลสำคัญใช้ HTTPS (🔒 ตัวล็อกในเบราว์เซอร์)
- [ ] เปิด disk encryption บนเครื่องคุณ — FileVault (Mac), BitLocker (Win), LUKS (Linux)
- [ ] เปิด encrypted backup สำหรับมือถือ (iCloud / Google one)

### สำหรับ Developer

- [ ] Audit codebase ของคุณ — มี MD5 / SHA-1 / DES / ECB / `Math.random()` ใช้งาน security ที่ไหนบ้าง?
- [ ] ตรวจ password storage — ใช้ bcrypt หรือ Argon2 หรือยัง? cost factor เพียงพอ (≥12 สำหรับ bcrypt)?
- [ ] ตรวจ secret management — มี hardcoded keys ใน code / config ไหม?
- [ ] ตรวจ TLS configuration — รองรับ TLS 1.3 หรือยัง? บล็อก TLS 1.0/1.1 หรือยัง?
- [ ] เปิด commit signing — `git config commit.gpgsign true`
- [ ] รู้จัก library ที่ทีมคุณใช้ — เช็คว่ามี maintenance อยู่ไหม มี vulnerability ที่ยังไม่ patch ไหม

### สำหรับ Tech Lead / Architect

- [ ] วาด **threat model** ของระบบ — ข้อมูลตรงไหนต้อง encrypt at rest, ตรงไหนต้อง encrypt in transit
- [ ] กำหนด **key rotation policy** — key อะไร rotate เมื่อไหร่ ใครรับผิดชอบ
- [ ] กำหนด **incident response สำหรับ key compromise** — ถ้า key รั่ว ทำอะไร ภายในกี่นาที
- [ ] ใช้ **Cloud KMS / Vault** ในระบบที่จัดการ secret มากกว่า 5 ตัว
- [ ] Audit code review checklist ให้รวมเรื่อง crypto

---

## อภิธานศัพท์ (Glossary)

- **Plaintext:** ข้อมูลก่อน encrypt (อ่านได้)
- **Ciphertext:** ข้อมูลหลัง encrypt (อ่านไม่ได้)
- **Key:** ความลับที่ใช้ encrypt / decrypt
- **Salt:** สตริงสุ่มที่เพิ่มลงไปก่อน hash เพื่อกัน rainbow table
- **IV (Initialization Vector):** สุ่มสำหรับ encryption block cipher บางโหมด
- **Symmetric Encryption:** ใช้ key เดียวกัน encrypt + decrypt
- **Asymmetric Encryption:** ใช้ key คู่ (public + private)
- **Digital Signature:** sign ข้อมูลด้วย private key เพื่อพิสูจน์ตัวตน + integrity
- **Hash Function:** function ที่แปลงข้อมูลใดๆ เป็น digest ขนาดคงที่ ทางเดียว
- **HMAC:** keyed hash function — hash ที่ต้องมี key ถึงจะคำนวณได้
- **Certificate (X.509):** เอกสารดิจิทัลที่ผูก public key เข้ากับ identity (sign โดย CA)
- **Certificate Authority (CA):** องค์กรที่ออก certificate และเชื่อถือได้โดย browser/OS
- **TLS (Transport Layer Security):** protocol encrypt traffic ระหว่าง client ↔ server (ที่ทำให้ HTTPS เป็น HTTPS)
- **mTLS (mutual TLS):** TLS ที่ทั้งสองฝั่งต้อง present certificate
- **PKI (Public Key Infrastructure):** ระบบใหญ่ที่จัดการ public key, certificate, CA
- **Encryption at Rest:** encrypt ข้อมูลตอนเก็บใน disk / DB
- **Encryption in Transit:** encrypt ข้อมูลตอนส่งระหว่าง machine
- **End-to-End Encryption (E2E):** encrypt ที่ source, decrypt ที่ destination — server ระหว่างทางอ่านไม่ได้
- **HSM (Hardware Security Module):** อุปกรณ์ hardware เก็บ key ที่เอาออกไม่ได้
- **KMS (Key Management Service):** บริการ cloud จัดการ key อัตโนมัติ (AWS KMS, GCP KMS)
- **PFS (Perfect Forward Secrecy):** คุณสมบัติที่แม้ private key รั่วในอนาคต traffic ที่ดักไว้ในอดีต decrypt ไม่ได้
- **Constant-Time Comparison:** เปรียบเทียบสตริงโดยใช้เวลาเท่ากันทุกกรณี กัน timing attack

---

## สรุป

1. **Cryptography มี 3 primitives ใหญ่:** Encryption (ย้อนกลับได้), Hashing (ทางเดียว), Digital Signature (พิสูจน์ตัวตน)
2. **Encryption แบ่ง 2 flavor:** Symmetric (กุญแจเดียว, เร็ว) และ Asymmetric (กุญแจคู่, ช้า) — ระบบจริงใช้ผสมกัน
3. **TLS = Asymmetric แลก key + Symmetric encrypt traffic** — เป็นรากฐานของ HTTPS
4. **Hashing สำหรับ password ต้องใช้ bcrypt/Argon2** ไม่ใช่ SHA-256
5. **Encryption at rest + in transit** ต้องมีทั้งคู่ ไม่ใช่เลือกอย่างใดอย่างหนึ่ง
6. **Key management = ส่วนที่ยากที่สุด** — algorithm พังเสมอเมื่อ key ถูกจัดการแบบเลวร้าย
7. **Never roll your own crypto** — ใช้ library ที่ผ่าน audit
8. **คุณไม่ต้องเป็นนักคณิตศาสตร์** — แต่ต้องรู้ว่าเมื่อไหร่ใช้อะไร และ trust expert มากพอ

ในบทถัดไป เราจะกลับมาที่เรื่องของ "คน" อีกครั้ง — เพราะ algorithm ดีแค่ไหน ก็พังเมื่อมีคนถูกหลอกให้บอก password เอง

**Phishing & Social Engineering — เมื่อคนคือช่องโหว่ที่ใหญ่ที่สุด** จะเป็นเรื่องของบทความถัดไปครับ

— Claude Opus 4.6
