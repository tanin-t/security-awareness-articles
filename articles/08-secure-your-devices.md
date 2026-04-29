# บทความที่ 8: Secure Your Devices — Laptop, Phone, และ อุปกรณ์ทุกชิ้นที่คุณใช้

**CyberSecurity Awareness Series — Part 4: Everyday Threats**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของ "ลืม Laptop ไว้ใน Taxi"

เดือนกุมภาพันธ์ ปี 2018 พนักงานของ **Coplin Health Systems** ในรัฐ West Virginia ลืม laptop ไว้ในรถ ลาน parking ของบริษัท — กลับมาอีกที laptop หายไป

ปัญหาที่ตามมา: laptop เครื่องนั้นมีข้อมูลคนไข้ของ Coplin **มากกว่า 43,000 คน** — ชื่อ, address, social security number, ประวัติการรักษา ฯลฯ — และ disk **ไม่ได้เข้ารหัส**

ผลที่ตามมา:

- Class action lawsuit จากคนไข้
- Settlement กับ HHS (Department of Health and Human Services) **2.175 ล้านดอลลาร์**
- Reputation damage ที่ตีค่าไม่ได้

[อ่านรายละเอียด](https://www.hipaajournal.com/coplin-health-systems-laptop-theft-2-175-million-settlement/)

ที่น่าตกใจคือ — **ถ้า laptop เครื่องนั้นเปิด disk encryption** (FileVault / BitLocker — ฟรีติดมากับ OS ทุกตัว) ปัญหานี้ก็คงจบในระดับ "พนักงานทำของหาย ซื้อใหม่"

แต่ในเมื่อ disk เปิดอ่านได้ — ใครจะเปิด laptop นั้นก็เข้าถึงข้อมูลทุกบาททุกสตางค์ได้

---

## เรื่องเล่าที่ใกล้ตัวกว่านั้น

ในประเทศไทย ผมจำเรื่องที่เพื่อนเล่าให้ฟังได้ — เขาเป็น dev ที่บริษัทแห่งหนึ่ง ใช้ MacBook ของบริษัท เผลอนั่งเล่นที่ร้านกาแฟแถว Asok ตอนกลางวัน เข้าห้องน้ำ 5 นาที กลับมา laptop หายไปแล้ว

ใน laptop มี:

- Source code ของ project ที่ทำอยู่
- AWS credentials ใน `~/.aws/credentials`
- SSH keys ของ production server
- Email ของบริษัท
- Slack workspace
- 1Password vault (encrypted แต่อยู่ในเครื่อง)

ทุกอย่างที่เขาใช้ทำงาน อยู่ในเครื่องเดียว — และเครื่องนั้นไปอยู่ในมือคนแปลกหน้า

โชคดีที่:

- เขามี **disk encryption เปิดไว้** → ใครเปิดเครื่องไม่ได้
- 1Password vault ต้องใช้ master password → safe
- เขา revoke AWS credentials + SSH keys ทันที → ไม่เกิดความเสียหาย
- บริษัทมี **MDM** (Mobile Device Management) → remote wipe ได้

ถ้าไม่มีสิ่งเหล่านี้ — เรื่องนี้จบไม่สวย แน่นอน

---

## ทำไม Device Security ถึงสำคัญกว่าที่คิด

ในโลกปี 2026 — อุปกรณ์ของคุณคือ **กระเป๋า + ตู้เซฟ + กุญแจ + บัตรประชาชน + บัตรเครดิต** รวมในชิ้นเดียว

ลองนึกดูครับ บนมือถือ/laptop คุณมี:

- **บัญชีธนาคาร** — login ผ่าน app, จำรหัสไว้
- **Email หลัก** — ที่ทุกอย่าง reset มาที่นี่
- **Password manager** — ที่เก็บรหัสผ่านทั้งชีวิต
- **2FA app** — ที่กันการเข้าระบบ
- **Notes ส่วนตัว** — รวมรหัส, recovery codes
- **Photos** — รูปถ่าย, screenshot ของบัตรประชาชน, passport
- **Chat** — LINE, WhatsApp, Telegram, Slack
- **GPS history** — Google Maps รู้ว่าคุณไปไหนทุกวัน
- **Source code** — สำหรับ dev
- **Production credentials** — สำหรับ DevOps / SRE

ถ้าใครเข้าถึงเครื่องคุณได้ = เข้าถึงทุกอย่างได้

นั่นคือเหตุผลว่าทำไม **device security** เป็นเรื่องพื้นฐานที่ ทุกคน ต้องทำให้ถูก — ไม่ใช่แค่ dev ไม่ใช่แค่ผู้บริหาร แค่คนที่มีสมาร์ทโฟนคนเดียวก็ใช่แล้ว

---

## ส่วนที่ 1: Disk Encryption — เปิดเดี๋ยวนี้ ฟรี

**Disk encryption** คือการ encrypt **ทั้ง disk** ของเครื่อง ดังนั้นถ้าใครเอา disk ออกไป เสียบเข้าเครื่องอื่น — อ่านอะไรไม่ได้เลย เห็นแต่ขยะ

### ทำไมต้องเปิด

โดย default — ถ้าเครื่องคุณ **ไม่มี** disk encryption แล้วใครได้เครื่องไป:

1. **เอา disk ออก** → เสียบเข้าเครื่องอื่น (ใช้ adapter ราคาไม่กี่ร้อยบาท)
2. **อ่านไฟล์ทั้งหมด** ผ่าน OS อื่น (Linux Live USB)
3. **Bypass login** — เพราะ password ของ Windows/Mac อยู่ใน disk เอง
4. **อ่าน password / API key / personal data** ทั้งหมด

ใช้เวลาเพียง 10-15 นาที ไม่ต้องมีทักษะอะไรพิเศษ มี YouTube tutorial หาดูได้

แต่ถ้า **disk encryption เปิด** — disk ที่ออกมาเห็นแต่ ciphertext ที่ไม่มี key ก็ decrypt ไม่ได้ในชาตินี้

### Disk Encryption ฟรีของแต่ละ OS

#### macOS — FileVault

```
System Settings → Privacy & Security → FileVault → Turn On
```

- ใช้ XTS-AES-128 หรือ XTS-AES-256
- Recovery key ต้องเก็บไว้ (จะถามตอนเปิด)
- เปิดได้ทันที background เริ่ม encrypt → ใช้งานต่อได้ปกติ

**สำคัญ:** เก็บ **recovery key** ไว้ในที่ปลอดภัย (password manager + กระดาษในตู้เซฟ) ไม่งั้นถ้าลืม login password = ข้อมูลหายตลอดกาล

#### Windows — BitLocker

```
Control Panel → System and Security → BitLocker Drive Encryption
หรือ Settings → Privacy & Security → Device Encryption
```

- ฟรีสำหรับ Windows 10/11 Pro / Enterprise
- Windows 11 Home มี **Device Encryption** (เวอร์ชันลด)
- ใช้ AES-128 หรือ AES-256
- ใช้ TPM (Trusted Platform Module) ถ้ามี — ฝัง key ใน hardware

**สำคัญ:** Recovery key เก็บใน Microsoft account หรือ AD (สำหรับเครื่องบริษัท)

#### Linux — LUKS

```bash
# ตั้งตอนติดตั้ง OS — เลือก "Encrypt the new <distro> installation for security"
# หรือใช้ cryptsetup กับ disk ใหม่:
sudo cryptsetup luksFormat /dev/sdX
sudo cryptsetup open /dev/sdX my_encrypted
sudo mkfs.ext4 /dev/mapper/my_encrypted
```

- มาตรฐานของ Linux
- ใช้ AES-XTS

#### มือถือ — เปิดมาแล้วโดย default

- **iPhone (iOS 8+):** เปิด disk encryption โดย default ทุกเครื่อง — ใช้ key ที่ผูกกับ passcode ของคุณ
- **Android (6.0+):** ส่วนใหญ่เปิดโดย default — ตรวจ Settings → Security → Encryption

**สำคัญสำหรับมือถือ:** ใช้ **passcode 6 หลักขึ้นไป** หรือดีกว่า — alphanumeric (เช่นใช้ "หนึ่งสองสามA!") เพราะ passcode 4 หลัก = brute force ใน 30 นาที

### ขั้นตอนการเปิด — Step by Step

#### macOS FileVault

1. **System Settings** (Cmd + Space → "System Settings")
2. **Privacy & Security** (sidebar)
3. Scroll หา **FileVault** → คลิก **Turn On**
4. Choose recovery method:
   - "Allow my iCloud account to unlock" — ง่าย แต่ผูกกับ iCloud
   - "Create a recovery key" — แนะนำ! เก็บ key ไว้เอง
5. **เก็บ recovery key** ใน password manager + พิมพ์ออกมาเก็บในตู้เซฟ
6. กด **Continue** → background encrypt เริ่มทำงาน
7. ใช้งานเครื่องได้ปกติ — encrypt 100% ภายใน 1-2 ชั่วโมง

#### Windows BitLocker (Pro/Enterprise)

1. Right-click drive ใน File Explorer → **Turn on BitLocker**
2. Choose "How do you want to back up your recovery key":
   - Save to Microsoft account
   - Save to a file (USB drive)
   - Print the recovery key
   - **เลือกอย่างน้อย 2 อย่าง**
3. Choose "Encrypt entire drive" (ปลอดภัยกว่า "Encrypt used disk space only")
4. Choose "New encryption mode" (XTS-AES)
5. **Run BitLocker system check** → restart
6. Encryption ทำงาน background

---

## ส่วนที่ 2: Screen Lock & Auto-Lock Timeout

นี่คือเรื่องที่ "ดูเหมือนน้อย" แต่สำคัญที่สุด

### ทำไมต้อง lock screen

ถ้า disk encryption ป้องกัน "เครื่องหาย" — screen lock ป้องกัน "เครื่องอยู่กับคุณ แต่คุณเดินไปหยิบกาแฟ"

ในระยะเวลา 30 วินาที ใครก็ตามที่ผ่านมาเครื่อง สามารถ:

- ส่ง email ในชื่อคุณ
- Copy ไฟล์ออก USB
- ติดตั้ง keylogger / RAT
- แชร์โพสต์น่าอับอายในชื่อคุณ
- เปิดหน้า browser ดู history

### กฎเหล็กของ Screen Lock

#### กฎที่ 1: ตั้ง auto-lock timeout สั้นๆ

| Device | Recommended timeout |
|---|---|
| Laptop ที่ทำงาน | 1-2 นาที |
| Laptop ส่วนตัวที่บ้าน | 5-15 นาที |
| มือถือ | 30 วินาที - 1 นาที |
| Desktop ที่ทำงาน | 5-10 นาที |

#### กฎที่ 2: Manual lock เป็นนิสัย

**ทุกครั้ง** ที่คุณลุกจากเครื่อง — แม้แต่ 30 วินาที — กด lock:

- **macOS:** `Ctrl + Cmd + Q`
- **Windows:** `Win + L`
- **Linux:** `Super + L` (ขึ้นอยู่กับ DE)
- **iPhone:** กดปุ่มข้าง 1 ครั้ง
- **Android:** กดปุ่ม power 1 ครั้ง

ฝึกให้เป็นนิสัยจน auto — เหมือนล็อกรถ

#### กฎที่ 3: Strong unlock method

- **Password / PIN** ขั้นต่ำ 8 ตัวอักษร
- **Biometric** (Touch ID / Face ID) เพิ่มความสะดวก
- **ห้าม** ใช้ pattern unlock 4 จุดที่เห็นได้ง่ายจากร่องนิ้ว

#### กฎที่ 4: Disable "Show preview" ใน notification

Lock screen ที่แสดง preview ของ message → คนเห็น OTP ของคุณได้โดยไม่ต้องปลดล็อก

- **iOS:** Settings → Notifications → Show Previews → "When Unlocked"
- **Android:** Settings → Notifications → Lock screen → "Hide content"

### ปุ่มลัด Quick Lock — ฝึกใช้

| Action | macOS | Windows | Linux (GNOME) |
|---|---|---|---|
| Lock screen | `Ctrl + Cmd + Q` | `Win + L` | `Super + L` |
| Sleep | `Option + Cmd + Power` | `Win + X → U → S` | varies |
| Force shutdown | hold Power 4s | hold Power 4s | hold Power 4s |

### Hot Corners (macOS) / Custom Hotkeys

ถ้าเป็น Mac — ตั้ง **Hot Corners** ให้มุมหน้าจอบางมุม trigger lock:

```
System Settings → Desktop & Dock → Hot Corners → 
   มุมที่เลือก → "Lock Screen"
```

โยนเมาส์ไปมุมนั้น = lock ทันที

---

## ส่วนที่ 3: Software Updates — ทำไมต้อง Update ทันที

หลายคนเลื่อน update ออกไปเพราะ:

- "ไม่อยาก restart"
- "กลัวจะมี bug ใหม่"
- "ใช้งานเครื่องอยู่"
- "ไว้ตอนเย็นค่อยทำ"

ทุกข้อข้างต้นมีเหตุผล — แต่ **ความเสี่ยงของการไม่ update มากกว่ามาก** ครับ

### Update คือการ Patch ช่องโหว่

Software ทุกตัวมี **bug** — บาง bug เป็น **vulnerability** (ช่องโหว่ที่ผู้โจมตีใช้ได้)

วงจร:

1. **นักวิจัย / hacker** เจอ vulnerability
2. **แจ้ง vendor** (responsible disclosure)
3. **Vendor ออก patch** + ออก CVE (Common Vulnerabilities and Exposures) record
4. **Patch ปล่อย** → ทุกคนต้อง update

ปัญหาคือ — ตั้งแต่ขั้นตอน 4 → ทุกผู้โจมตีในโลก รู้ว่ามีช่องโหว่นี้อยู่ และรู้วิธี exploit

ถ้าคุณ **ไม่ update** — คุณคือเป้านิ่งสำหรับการโจมตีที่รู้กันทั่วโลกแล้ว

### Patch Tuesday

**Microsoft** ออก security update ทุก **Tuesday ที่สอง ของเดือน** — เรียกว่า "Patch Tuesday"

วงการ security ก็เรียกวันต่อมา (Wednesday) ว่า **"Exploit Wednesday"** — เพราะตั้งแต่ Patch ออก hacker reverse engineer patch หา vulnerability แล้วเริ่มยิง user ที่ยังไม่ update

ในเฉลี่ย — **ผู้โจมตีเริ่ม exploit ภายใน 24-48 ชั่วโมง** หลัง patch ออก

### Zero-Day Vulnerability

**Zero-day** = vulnerability ที่ผู้โจมตีรู้ ก่อนที่ vendor จะรู้ → ยังไม่มี patch

ในขณะที่ vendor กำลัง develop patch — ผู้โจมตีใช้ช่องโหว่นี้ได้เต็มที่

ตัวอย่าง zero-day ที่ดังที่สุด:

- **EternalBlue** (2017) — Microsoft Windows SMB เป็นช่องทางของ WannaCry ransomware ที่ infect 300,000 เครื่องใน 150 ประเทศ
- **Log4Shell** (2021) — Apache Log4j มี vulnerability RCE ที่กระทบทุก enterprise ใน 24 ชั่วโมงที่เปิดเผย
- **Pegasus** (2020-) — NSO Group spyware ใช้ zero-day ใน iOS/Android โจมตีนักข่าว นักการเมืองทั่วโลก

ความจริงคือ — **คุณป้องกัน zero-day ที่ยังไม่รู้ไม่ได้** แต่คุณป้องกัน "1-day" (vulnerability ที่ patch แล้ว แต่ยังไม่ติดตั้ง) ได้ — โดย **update ทันที**

### Update Strategy ที่แนะนำ

#### สำหรับเครื่องส่วนตัว

- **OS updates** → install ภายใน 24-48 ชั่วโมง
- **Browser updates** → install ทันที (Chrome / Firefox auto-update อยู่แล้ว)
- **Apps สำคัญ** (1Password, Slack, etc.) → install ทันที
- **Apps ทั่วไป** → ตั้ง auto-update

#### สำหรับเครื่องบริษัท

- **Critical patch** → 24-72 ชั่วโมง
- **High severity** → 1 สัปดาห์
- **Medium** → 1 เดือน
- **Low** → quarterly

หลัก: **ความเร่งด่วน ขึ้นกับความรุนแรงของ vulnerability** — บริษัทที่จริงจังจะมีนโยบาย patch management

#### สำหรับมือถือ

- **iOS:** Settings → General → Software Update → Automatic Updates → ON
- **Android:** Settings → System → System Update → Auto-update

### "ฉันใช้เครื่องเก่าที่ vendor หยุด support แล้ว"

ถ้า OS / device ของคุณ **end of life** (no more security update):

- **Windows 7:** EOL ตั้งแต่ 2020 — **อย่าใช้กับ internet**
- **Windows 10:** EOL ตุลาคม 2025
- **macOS Big Sur:** EOL ปลาย 2023
- **iPhone 6/6s:** ไม่ได้รับ iOS update ตั้งแต่ 2020
- **Android phones:** ส่วนใหญ่ EOL หลัง 2-3 ปี (ยกเว้น Pixel, iPhone)

ถ้าเป็น EOL device → **อัพเกรด** หรือ **ใช้แค่ offline เท่านั้น** ไม่งั้นคือเป้านิ่ง

### Application Updates ก็สำคัญ

ไม่ใช่แค่ OS — apps ที่คุณใช้ทุกวันก็มี vulnerability:

- **Browser:** Chrome, Firefox, Safari, Edge — สำคัญที่สุด เพราะคุณเอาไปเปิดเว็บอันตราย
- **Office suite:** Microsoft Office, Google Workspace
- **PDF reader:** Adobe Acrobat (โดยเฉพาะ — มีประวัติ vulnerability เยอะ)
- **Java / .NET runtime** ถ้ามี
- **Communication apps:** LINE, WhatsApp, Slack, Zoom

ใช้ **package manager** ช่วย:

- **macOS:** Homebrew (`brew upgrade`), App Store auto-update
- **Windows:** Winget, Chocolatey, หรือ Microsoft Store
- **Linux:** apt, dnf, pacman — auto-update ได้

---

## ส่วนที่ 4: Public WiFi Risks & VPN

ร้านกาแฟ, สนามบิน, โรงแรม, mall — มี free WiFi ที่ทุกคนใช้

แต่... ปลอดภัยจริงไหม?

### ความเสี่ยงของ Public WiFi

#### 1. Man-in-the-Middle (MITM) Attack

ผู้โจมตีตั้ง WiFi ที่หน้าตาเหมือน WiFi ของร้าน:

```
Real WiFi:    "Starbucks_Free_WiFi"
Fake WiFi:    "Starbucks_Free_WiFi" (← ตัวเดียวกัน, signal แรงกว่า)
```

คุณ connect — traffic ผ่านเครื่องของผู้โจมตี เขาอ่าน/แก้ทุกอย่างได้

#### 2. Packet Sniffing

WiFi เก่าๆ ที่ไม่มี encryption (open network) → ทุกคนใน network เดียวกัน อ่าน traffic ของกันได้ด้วยเครื่องมือ free เช่น Wireshark

#### 3. Evil Twin

เหมือน MITM แต่ลึกกว่า — fake WiFi มี captive portal ที่หลอกขอรหัสผ่าน

#### 4. Karma Attack

มือถือคุณ "remember" WiFi เก่าๆ — เห็น SSID เดียวกัน auto connect ไปเลย ผู้โจมตีตั้ง WiFi ที่ใช้ SSID ทั่วไป (`@WiFi`, `Free_WiFi`, `Starbucks WiFi`) → มือถือ connect เอง

### TLS/HTTPS ช่วยได้ — แต่ไม่หมด

ถ้าเว็บที่คุณใช้เป็น HTTPS — ผู้โจมตีอ่าน content ไม่ได้ (ดูบทที่ 6 เรื่อง TLS)

แต่ — เขายังเห็น:

- **DNS query** — รู้ว่าคุณเข้าเว็บอะไร (แม้ HTTPS)
- **Metadata** — IP ปลายทาง, traffic pattern, timing
- **เว็บที่ไม่ใช่ HTTPS** — เห็นทุกอย่าง

### VPN เข้ามาแก้ปัญหานี้

**VPN (Virtual Private Network)** สร้าง encrypted tunnel ระหว่างเครื่องคุณ → VPN server

```
[คุณ] ─ encrypted tunnel ─→ [VPN server] ─→ [internet]
```

ผู้โจมตีใน WiFi เห็นแค่ "traffic ที่ encrypt ไป VPN server" — ไม่เห็น content, ไม่เห็นว่าคุณเข้าเว็บไหน

### VPN ที่แนะนำ

#### สำหรับ privacy ส่วนตัว

- **Mullvad** — ไม่เก็บ log, ไม่ต้องสมัคร email, จ่ายเงินสดได้
- **ProtonVPN** — open source, free tier มี
- **IVPN** — focus privacy, audited

#### สำหรับใช้งานทั่วไป

- **NordVPN** — server เยอะ ใช้ง่าย
- **ExpressVPN** — ความเร็วดี

**❌ หลีกเลี่ยง:**
- VPN ฟรีๆ ที่ไม่รู้ที่มา (เช่น Hola, SuperVPN) — มีประวัติขายข้อมูล user
- VPN ของบริษัทจีนหรือรัสเซีย ที่ต้องส่งข้อมูลให้รัฐบาล

### VPN ขององค์กร

ถ้าคุณทำงานบริษัท — บริษัทมักจะมี **corporate VPN**

```
[คุณที่ café] → [Corporate VPN] → [Internal company resources]
```

ใช้สำหรับเข้าระบบภายใน + ป้องกัน WiFi attacks

ในปี 2026 หลายบริษัทย้ายไป **Zero Trust Network Access (ZTNA)** แทน VPN เก่า — หลักการเดียวกัน แต่ granular กว่า (จะคุยในบทที่ 19)

### ทางเลือกแทน VPN

ถ้าไม่อยากจ่าย VPN:

#### 1. Mobile Hotspot

ใช้ **มือถือเป็น hotspot** แทนใช้ public WiFi:

- ปลอดภัยกว่ามาก (data ของ telco)
- ไม่ฟรีถ้า data หมด
- ใช้แบตเร็ว

#### 2. SSH Tunnel

ถ้าคุณมี server ที่ไหนสักแห่ง:

```bash
ssh -D 1080 user@your-server.com
# จากนั้นตั้ง SOCKS proxy ใน browser → 127.0.0.1:1080
```

#### 3. Cloudflare WARP

ฟรี, app ง่าย — encrypt DNS + traffic บางส่วน
ไม่ใช่ VPN เต็มรูป แต่ดีกว่าไม่มี

---

## ส่วนที่ 5: Physical Security — สิ่งที่ Tech Skills แก้ไม่ได้

Disk encryption ดี, software update ครบ, VPN เปิด — ทุกอย่างอ่อนแรงเมื่อ **เครื่องอยู่กับคนอื่น**

### กฎพื้นฐาน

#### กฎที่ 1: ห้ามทิ้ง laptop ไว้ตามลำพัง

แม้แต่ในร้านกาแฟ ห้องประชุม โต๊ะทำงานในออฟฟิศ — เก็บไป หรือเอาไปด้วยทุกครั้งที่ลุก

#### กฎที่ 2: ใช้ kensington lock เมื่อต้องทิ้งไว้

มี cable lock ที่คล้องโต๊ะ — ราคาไม่กี่ร้อยบาท ทำให้ขโมยลำบาก

#### กฎที่ 3: แต่ก็ไม่เชื่อใจ

ใน airport / hotel safe / luggage storage — ถ้าจำเป็นต้องทิ้งไว้ ให้มั่นใจว่า:

- **Lock เครื่องเสมอ**
- **Disk encryption เปิด**
- **มีข้อมูล sensitive น้อยที่สุด**
- **บันทึก serial number** ไว้สำหรับแจ้งหาย

#### กฎที่ 4: ระวัง shoulder surfing

ที่ café / airport — คนข้างๆ มองเห็นจอคุณ:

- ใช้ **privacy filter** บนหน้าจอ — มองเฉียงไม่เห็น
- ระวังคน "เดินผ่าน" ตอนคุณกรอกรหัสผ่าน
- หลังคุณเข้าเว็บการเงิน — ปิดหน้าก่อนลุก

#### กฎที่ 5: USB Port Security

อย่า:

- ใช้ USB charger ในที่ไม่รู้จัก (อาจเป็น **juice jacking** — USB ที่อ่าน data)
- เสียบ USB drive ที่ไม่รู้ว่ามาจากไหน (**USB drop attack** — แฮกเกอร์ทิ้ง USB ที่ฝัง malware ใน parking lot บริษัท หวังว่าใครจะหยิบไปเสียบ)

ใช้ **USB data blocker** (USB condom) — ตัวเล็กๆ ที่ปิด data pin ปล่อยแต่ power ผ่าน — สำหรับ charger สาธารณะ

#### กฎที่ 6: Webcam Cover

ติด **physical sticker** หรือ **webcam slider** ปิดกล้อง laptop เมื่อไม่ใช้:

- ราคาไม่กี่บาท — แต่กัน RAT (Remote Access Trojan) ที่อาจแอบเปิดกล้อง
- Mark Zuckerberg, FBI Director เคยถ่ายรูปกับ laptop ที่มี tape ปิดกล้อง

#### กฎที่ 7: Microphone

แอป malicious เปิด mic ฟังคุณได้:

- ใน System settings — block app ที่ไม่จำเป็นไม่ให้เข้าถึง mic
- บางคนใช้ **physical mic blocker** (jack ที่ปิดวงจร mic)

### Travel Security

เวลาเดินทาง — เพิ่มความระวัง:

#### ก่อนออกเดินทาง

- **Backup เครื่อง** ครบก่อนเดินทาง
- **Update OS** ทุกตัว
- **Decrypt sensitive data** ที่ไม่จำเป็นต้องเดินทางด้วย — ใส่ encrypted USB ที่ทิ้งไว้
- **เปลี่ยนรหัสผ่าน** สำคัญก่อนเดินทาง (เพราะถ้าโดนเขาขาย ก็เพิ่งใช้ได้ short window)

#### ระหว่างเดินทาง

- **เครื่อง laptop ใน carry-on เสมอ** ห้ามใส่ checked baggage
- **ใช้ Mobile hotspot** แทน public WiFi
- **Power down เครื่อง** เมื่อผ่านด่านตรวจ (encryption key ออกจาก RAM)
- **อย่าออกจากเครื่องเมื่อ unlock อยู่**

#### หลังกลับ

- **Scan เครื่อง** ดู malware
- **เช็ค activity log** ของบัญชีทุกตัว
- **ถ้ามีโอกาส tampering** (เช่น เครื่องอยู่กับเจ้าหน้าที่ตรวจคนเข้าเมืองนานๆ) — สงสัยให้สงสัยที่สุด อาจ format + reinstall

---

## ส่วนที่ 6: Mobile Device Management (MDM) สำหรับทีม

ถ้าคุณเป็น **Tech Lead / IT Lead / DevOps** ที่ดูแลทีม — เรื่อง MDM สำคัญ

### MDM คืออะไร

**MDM (Mobile Device Management)** คือระบบที่ให้บริษัท:

- **Enforce policy** บนเครื่องของพนักงาน (passcode, encryption, etc.)
- **Push apps** จากระยะไกล
- **Remote wipe** ถ้าเครื่องหาย
- **Audit + monitor** การใช้งาน
- **Block apps** ที่ไม่ควรใช้
- **Containerize** data ของบริษัท แยกจากของส่วนตัว (BYOD)

### MDM Solutions ที่นิยม

| Tool | Best for |
|---|---|
| **Microsoft Intune** | บริษัทที่ใช้ Microsoft 365 |
| **Jamf Pro** | บริษัทที่ Apple-heavy |
| **Kandji** | Apple ecosystem, modern UX |
| **MobileIron / Ivanti** | Enterprise ใหญ่ |
| **Google Workspace** | บริษัทที่ใช้ Google Workspace |
| **Hexnode** | Mid-size, multi-OS |
| **Workspace ONE** | VMware-based, multi-platform |

### Policy ที่ควรบังคับใช้ผ่าน MDM

#### บังคับเสมอ:

- ✅ **Passcode** ขั้นต่ำ (6 หลัก alphanumeric)
- ✅ **Auto-lock timeout** (1-2 นาที)
- ✅ **Disk encryption** (FileVault / BitLocker)
- ✅ **OS updates** auto + force ภายใน X วันหลัง release
- ✅ **Antivirus / EDR** (เช่น CrowdStrike, SentinelOne)
- ✅ **VPN** auto-connect เมื่ออยู่นอกออฟฟิศ
- ✅ **Block USB storage** (ป้องกัน data exfiltration)
- ✅ **Remote wipe capability**
- ✅ **Compliance report** — เครื่องที่ไม่ผ่าน policy ถูก block จาก resource บริษัท

#### แนะนำเพิ่ม:

- ⚠️ Browser extension allowlist
- ⚠️ App allowlist (เครื่องบริษัทเท่านั้น)
- ⚠️ Network connection logging
- ⚠️ Geo-fencing (alert ถ้าเครื่องออกประเทศ)

### BYOD vs Corporate-owned

#### BYOD (Bring Your Own Device)

- พนักงานใช้เครื่องส่วนตัว access ระบบบริษัท
- ความเสี่ยง: privacy ของพนักงาน vs security ของบริษัท
- Solution: **container/profile** ที่แยก data — บริษัท wipe เฉพาะ profile ได้ ไม่กระทบ data ส่วนตัว

#### Corporate-owned

- บริษัทเป็นเจ้าของเครื่อง — full control
- ปลอดภัยกว่า — แต่ต้นทุนสูงกว่า
- เหมาะกับ role ที่ access sensitive data

### Onboarding / Offboarding

#### Onboarding (พนักงานใหม่)

- บัญชีใหม่ pre-configured ผ่าน MDM
- เครื่อง enroll automatic เมื่อเปิดครั้งแรก
- Apps install อัตโนมัติ

#### Offboarding (พนักงานออก)

- **วันที่สุดท้าย** — disable account ทุกระบบ
- **Wipe corporate data** จากเครื่องทันที
- ถ้าเครื่องบริษัท → recall เครื่อง
- Audit access log ดูว่า download อะไรออกในสัปดาห์สุดท้าย

---

## Hands-on: Security Checklist สำหรับเครื่องทุกชนิด

### Laptop / Desktop

- [ ] **Disk encryption เปิด** (FileVault / BitLocker / LUKS)
- [ ] **Recovery key เก็บปลอดภัย** (password manager + กระดาษในตู้เซฟ)
- [ ] **Login password ยาว** (ขั้นต่ำ 12 ตัวอักษร)
- [ ] **Auto-lock timeout** ≤ 5 นาที
- [ ] **OS up-to-date** (เช็ค monthly)
- [ ] **Browser auto-update** เปิดไว้
- [ ] **Antivirus** ติดตั้ง + active
- [ ] **Firewall** เปิด (built-in OS firewall)
- [ ] **Webcam cover** ติดเมื่อไม่ใช้
- [ ] **Backup** อย่างน้อย weekly (Time Machine / Windows Backup / cloud)
- [ ] **Disable auto-login** — ต้องใส่ password ทุกครั้งที่เปิดเครื่อง
- [ ] **Find My** เปิดไว้ (Apple) / Windows Find My Device

### มือถือ (iPhone / Android)

- [ ] **Passcode 6+ หลัก** (alphanumeric ดียิ่งกว่า)
- [ ] **Auto-lock 30s - 1 นาที**
- [ ] **Biometric** เปิด (Face/Touch ID)
- [ ] **Lock screen notification preview** = "When Unlocked" / "Hide content"
- [ ] **iOS / Android version ล่าสุด**
- [ ] **App auto-update** เปิด
- [ ] **Find My iPhone / Find My Device** เปิด
- [ ] **Lockdown Mode** (iOS) เปิดถ้าเป็นเป้าหมายเฉพาะ
- [ ] **Backup** เปิด (iCloud / Google Cloud) — encrypted
- [ ] **App permissions** ตรวจเป็นระยะ (ลบ permission ที่ไม่จำเป็น)
- [ ] **Hide WiFi list** จาก notification
- [ ] **Remove auto-connect** ของ public WiFi เก่าๆ

### Network / WiFi

- [ ] **Home router admin password** เปลี่ยนจาก default
- [ ] **Router firmware update** เป็นระยะ
- [ ] **WPA3** หรือ WPA2 (ห้าม WEP, ห้าม open)
- [ ] **WPS disabled**
- [ ] **Guest network** แยกสำหรับแขก / IoT
- [ ] **VPN service** subscription (สำหรับ travel + public WiFi)
- [ ] **DNS-over-HTTPS** เปิด (Cloudflare 1.1.1.1, Quad9)

### Travel Kit

- [ ] **USB data blocker** อย่างน้อย 2 ตัว
- [ ] **Privacy filter** สำหรับ laptop screen
- [ ] **Kensington lock**
- [ ] **Encrypted USB drive** สำหรับ backup
- [ ] **Power bank** (เพื่อไม่ต้องใช้ public charger)
- [ ] **Encrypted backup ก่อนเดินทาง**

---

## Action Items

### วันนี้

- [ ] เปิด **disk encryption** บน laptop ของคุณ — ถ้ายังไม่ได้เปิด
- [ ] ตั้ง **auto-lock timeout 5 นาที** (laptop) / 30 วินาที (มือถือ)
- [ ] เช็ค OS version → update ถ้ามี

### สัปดาห์นี้

- [ ] **Backup recovery key** ของ disk encryption ไว้ในที่ปลอดภัย
- [ ] **ติด webcam cover** บน laptop
- [ ] **ซื้อ USB data blocker** ขำๆ ไว้ใช้
- [ ] **เลือก VPN service** สำหรับ travel
- [ ] **Audit app permissions** บนมือถือ — ลบของไม่จำเป็น
- [ ] **Remove old WiFi networks** จากมือถือ

### สำหรับ Tech Lead

- [ ] ประเมินว่าทีมต้องใช้ **MDM** หรือไม่
- [ ] เลือก MDM solution ที่เหมาะกับ stack
- [ ] เขียน **security baseline policy** สำหรับเครื่องทีม
- [ ] Setup **patch management process**
- [ ] กำหนด **incident response** สำหรับ "เครื่องหาย"
- [ ] **Training session** ทีมเรื่อง physical security

---

## บทเรียนชีวิตจากบทความนี้

> **ดูแลเครื่องมือของคุณ — เหมือนดูแลรถยนต์ ต้อง maintain ไม่ใช่แค่ใช้**

ลองคิดเรื่องรถยนต์ดูครับ:

- ตอนซื้อรถ — ใหม่เอี่ยม สวย เร็ว
- ปีแรก — ทุกอย่างปกติ
- ปีที่สอง — เริ่มมี noise เล็กน้อย แต่ขับได้
- ปีที่สาม — น้ำมันเครื่องไม่เปลี่ยน, ยางเริ่มเสื่อม, brake pad บาง
- ปีที่ห้า — รถเสียบ่อย ค่าซ่อมแพง สุดท้ายขายขาดทุน

ขณะที่อีกคนหนึ่ง:

- เปลี่ยนน้ำมันเครื่องทุก 10,000 km
- เช็คแรงดันยางทุกเดือน
- ส่งซ่อมตามตารางที่บริษัทแนะนำ
- ทำความสะอาดเป็นประจำ
- รถใช้ได้ 15 ปี — ไม่มีปัญหาใหญ่

**ความต่างไม่ใช่ "ใครได้รถดีกว่า" — แต่เป็น "ใครดูแลดีกว่า"**

อุปกรณ์ดิจิทัลของคุณก็เหมือนกันครับ:

- **Update เป็นน้ำมันเครื่อง** — ไม่ทำ → engine พัง → กลายเป็นเป้าโจมตี
- **Disk encryption เป็นการ lock รถ** — ไม่ทำ → ใครก็ขับเอาไปได้
- **Backup เป็นประกัน** — ไม่มี → เกิดอะไรขึ้น = หมดตัว
- **MDM เป็นระบบ alarm** — สำหรับองค์กร = early warning + remote control
- **Physical security เป็นการขับขี่ปลอดภัย** — เทคโนโลยีดีแค่ไหน ก็แก้ "ทิ้งกุญแจคารถ" ไม่ได้

> **เครื่องมือดี + คนใช้ที่ดูแลไม่เป็น = ใช้ได้ไม่นาน**  
> **เครื่องมือธรรมดา + คนใช้ที่ดูแลดี = ใช้ได้ยาวนาน + ปลอดภัย**

ในชีวิตทุกเรื่องก็เช่นกัน:

- **สุขภาพ** — ไม่ใช่ "ตอนป่วยรักษา" แต่ "ตอนปกติดูแล"
- **ความสัมพันธ์** — ไม่ใช่ "ตอนทะเลาะแก้" แต่ "ตอนปกติใส่ใจ"
- **อาชีพ** — ไม่ใช่ "ตอนถูกไล่ออกหางาน" แต่ "ตอนยังมีงานพัฒนาตัวเอง"
- **เงิน** — ไม่ใช่ "ตอนเดือดร้อนกู้" แต่ "ตอนปกติออม"

**Maintenance** เป็น mindset ที่สำคัญมาก — ไม่หรูหรา ไม่น่าตื่นเต้น แต่เป็นความต่างระหว่าง "อยู่รอด" กับ "ไม่อยู่รอด" ในระยะยาว

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

---

## อภิธานศัพท์ (Glossary)

- **Disk Encryption:** เข้ารหัสทั้ง disk เพื่อป้องกันการอ่านเมื่อ disk ออกจากเครื่อง
- **FileVault:** disk encryption built-in ของ macOS
- **BitLocker:** disk encryption built-in ของ Windows Pro/Enterprise
- **LUKS (Linux Unified Key Setup):** disk encryption มาตรฐานของ Linux
- **TPM (Trusted Platform Module):** chip บน motherboard ที่เก็บ encryption key
- **Recovery Key:** key สำรองสำหรับปลดล็อก disk encryption ถ้าลืม password
- **MDM (Mobile Device Management):** ระบบ centralized จัดการอุปกรณ์ในองค์กร
- **EDR (Endpoint Detection and Response):** antivirus รุ่นใหม่ที่ detect + respond แบบ real-time
- **Patch Tuesday:** วันที่ Microsoft ออก security update รายเดือน (Tuesday ที่ 2)
- **Zero-day Vulnerability:** ช่องโหว่ที่ผู้โจมตีรู้ก่อน vendor — ยังไม่มี patch
- **CVE (Common Vulnerabilities and Exposures):** ฐานข้อมูล vulnerability ระดับโลก
- **MITM (Man-in-the-Middle):** attack ที่ผู้โจมตีอยู่ระหว่าง 2 ฝ่ายที่กำลังสื่อสาร
- **Evil Twin:** WiFi access point ปลอมที่เลียนแบบ legitimate AP
- **Karma Attack:** WiFi attack ที่อาศัย client device "remember" SSID เก่า
- **Juice Jacking:** USB charger สาธารณะที่ขโมย data
- **USB Drop Attack:** ผู้โจมตีทิ้ง USB drive ที่ฝัง malware เพื่อให้คนหยิบไปเสียบ
- **Privacy Filter:** ฟิล์มกรองหน้าจอที่ทำให้ดูเฉียงไม่เห็น
- **Kensington Lock:** cable lock สำหรับล็อก laptop กับโต๊ะ
- **BYOD (Bring Your Own Device):** policy ที่อนุญาตให้พนักงานใช้เครื่องส่วนตัว
- **VPN (Virtual Private Network):** encrypted tunnel ระหว่าง device → server กลาง
- **ZTNA (Zero Trust Network Access):** modern alternative ของ VPN — verify ทุก request

---

## สรุป

1. **Disk encryption เปิดเดี๋ยวนี้** — ฟรีติด OS ทุกตัว เปิดเลย เก็บ recovery key ดี
2. **Screen lock + auto-lock timeout สั้นๆ** — กฎพื้นฐานที่ทุกคนต้องทำ
3. **Software update ทันที** — patch Tuesday → Exploit Wednesday → ไม่ update = เป้านิ่ง
4. **Public WiFi อันตราย** — ใช้ VPN, mobile hotspot, หรือ HTTPS เท่านั้น
5. **Physical security ไม่มี tech แทน** — Kensington lock, webcam cover, USB data blocker
6. **MDM สำหรับทีม** — central control + remote wipe + compliance enforcement
7. **Maintenance mindset** — ดูแลอุปกรณ์เหมือนดูแลรถ ป้องกันดีกว่ารักษา

ในบทถัดไป เราจะเข้าเรื่อง **พฤติกรรมออนไลน์** — Safe Browsing, ad-blocker, browser extension, cookie management และอื่นๆ ที่คุณทำได้ทุกวันเพื่อป้องกันตัวเองในโลกออนไลน์

— Claude Opus 4.6
