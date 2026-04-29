# บทความที่ 9: Safe Browsing & Online Hygiene — พฤติกรรมออนไลน์ที่ปลอดภัย

**CyberSecurity Awareness Series — Part 4: Everyday Threats**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของ Browser Extension ที่ "ขโมยทุกอย่าง"

เดือนกรกฎาคม ปี 2023 บริษัท security ชื่อ **Avast** ออกรายงานเปิดเผยเรื่องที่ทำให้ user หลายล้านคนตกใจครับ — มี **browser extension ใน Chrome Web Store** ที่ดาวน์โหลดรวมกัน **มากกว่า 75 ล้านครั้ง** ถูกพบว่าเป็น **malicious extension** ที่:

- ขโมย session cookie ของ Facebook
- บันทึก keystroke ของ user
- ส่งข้อมูล browsing history กลับไปยัง server ในต่างประเทศ
- บางตัวแอบ install adware เพิ่ม

extension เหล่านี้มีชื่อน่าเชื่อถือ เช่น "Netflix Party", "Quick Translate", "PDF Toolkit Pro" — มีรีวิว 4-5 ดาวเป็นพันๆ และ Google ก็ approve ขึ้น store แล้ว

[อ่านรายละเอียดที่ Bleeping Computer](https://www.bleepingcomputer.com/news/security/malicious-chrome-extensions-with-75m-installs-removed-from-web-store/)

ที่น่าตกใจคือ — **ผู้ใช้ทั่วไปไม่มีทางรู้** ว่า extension ไหนปลอดภัย เพียงแค่เห็นชื่อบนหน้า Chrome Web Store และดูว่ามีคน install เยอะแล้วก็ติดตั้งไปเลย

ไม่ใช่ไอเดียบ้าๆ ครับ — เพราะระบบของ Chrome / Firefox / Edge ออกแบบมาให้ extension มี permission **มหาศาล** เช่น "อ่าน + แก้ไข data ของทุกเว็บที่คุณเข้า" — extension แค่ตัวเดียวที่ใจร้าย ก็เท่ากับ **มีคนนั่งดูข้างๆ คุณตอนใช้คอม**

นี่คือเรื่องที่บทความนี้จะแก้ไขครับ — **online hygiene** หรือ "พฤติกรรมออนไลน์ที่ปลอดภัย" ไม่ใช่แค่ "ใช้ HTTPS" แต่รวมถึง:

- **Browser extensions** ที่คุณติดตั้ง
- **Cookies & tracker** ที่ตามคุณไป
- **Permissions** ที่คุณกด "Allow" โดยไม่อ่าน
- **Email + identity** ที่กระจายอยู่ทุกที่

มาดูกันทีละเรื่องครับ

---

## ส่วนที่ 1: Browser Extensions — ของดีที่กลับร้าย

### ทำไม Extension ถึงอันตราย

เมื่อคุณ install browser extension — มันมักจะขอ permission ประมาณนี้:

```
Allow this extension to:
✓ Read and change all your data on all websites you visit
✓ Read your browsing history
✓ Display notifications
✓ Manage your downloads
```

นี่ไม่ใช่ permission พิเศษ — นี่คือ **default** ของ extension ส่วนใหญ่ที่ขายของได้

ความจริงที่น่ากลัว:

- Extension เห็น **ทุก URL** ที่คุณเปิด
- เห็น **ทุก content** ในหน้าเว็บ (รวม email, password ที่คุณกรอก, คำสั่งซื้อ ที่คุณดู)
- **แก้ไข** content ได้ (เช่น แอบเปลี่ยนเลขบัญชีรับโอนเงิน)
- ส่งข้อมูล **กลับไป server ของผู้พัฒนา** ได้

เปรียบเทียบ — นี่เหมือนคุณจ้างคนมาเป็น "เลขา" ที่นั่งข้างคุณตลอดเวลา เห็นทุกอย่าง อ่านทุก email พิมพ์ทุก password — แค่ขอชื่อให้น่ารักหน่อยก็ผ่าน

### Extension ที่ "ปลอดภัย" สามารถกลายเป็นอันตรายได้

นี่คือสิ่งที่หลายคนไม่รู้ครับ — **extension ที่เคยปลอดภัย กลายเป็น malicious ได้** ผ่านวิธีต่างๆ:

#### 1. Sold to Bad Actors

ผู้พัฒนาเดิมขาย extension ให้บริษัทอื่น — บริษัทใหม่อาจเปลี่ยน code:

ตัวอย่าง: **The Great Suspender** — extension popular ที่มี user หลายล้าน ถูกซื้อต่อในปี 2020 — เจ้าใหม่ใส่ malicious code เข้าไป Google ต้องลบจาก store ในปี 2021

#### 2. Hacked Update Mechanism

ผู้โจมตีแฮก dev account ของ extension แล้ว push update ที่มี malware:

ตัวอย่าง: **Edit This Cookie**, **Web Developer**, **Hover Zoom** ทั้งหมดเคยถูก hijack แบบนี้

#### 3. Dependency Compromise

Extension ใช้ library ภายนอก — library นั้นถูกแฮก → extension ก็ติดไปด้วย

### กฎเหล็กของ Browser Extensions

#### กฎที่ 1: ใช้ extension น้อยที่สุด

ทุก extension เป็น **attack surface** เพิ่ม → ยิ่งน้อย ยิ่งปลอดภัย

ลองถามตัวเอง:
- "Extension นี้คุ้มกับความเสี่ยงไหม?"
- "ถ้ามันหายไปจะกระทบอะไร?"

#### กฎที่ 2: ตรวจ permission ก่อนติดตั้ง

ก่อนกด "Add to Chrome":

- อ่าน "Permissions" ในหน้า extension
- ถ้า extension แค่ "เปลี่ยนสีปุ่ม" ทำไมต้องขอ "read all data on all sites"?
- ถ้า permission ไม่สมเหตุสมผล → อย่าติดตั้ง

#### กฎที่ 3: Source ของ Extension

ติดตั้งจาก:
- ✅ **Chrome Web Store** (Chrome) / **Firefox Add-ons** / **Edge Add-ons**
- ❌ **ไม่เคย** ติดตั้งจากเว็บอื่น (เว้นแต่ enterprise extension จากบริษัท)
- ❌ **ไม่เคย** drag-and-drop file `.crx` ที่หามาจาก internet

#### กฎที่ 4: ดู author + reviews

- Author เป็น org ที่น่าเชื่อถือไหม? (เช่น "Google LLC", "1Password", etc.)
- มี link ไปเว็บ official ไหม?
- รีวิวมีคนบ่นเรื่องผิดปกติไหม?
- จำนวน install + อายุของ extension เก่าแค่ไหน?

#### กฎที่ 5: ตรวจสอบเป็นระยะ

ทุก 3-6 เดือน เปิดหน้า extension ของคุณ:

- มีตัวที่ไม่ได้ใช้แล้วไหม → ลบ
- มีตัวที่ดู permission แปลกๆ ไหม → ลบ
- มีตัวที่ developer หายไปแล้วไหม → ลบ

#### กฎที่ 6: ระวัง "Permission Changes"

บางครั้ง extension จะขอ permission เพิ่มผ่าน update — Chrome จะแจ้งเตือน:

```
"Extension X has been updated and is requesting new permissions"
```

อ่านอย่างละเอียดก่อนกด "Allow"

### Extensions ที่ผมแนะนำให้มี

#### Security & Privacy

- **uBlock Origin** ([Chrome](https://chromewebstore.google.com/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm) / [Firefox](https://addons.mozilla.org/firefox/addon/ublock-origin/)) — ad blocker + tracker blocker ที่ดีที่สุด open source
- **Privacy Badger** by EFF — block tracker ที่ฉลาด
- **Bitwarden / 1Password** — password manager
- **HTTPS Everywhere** (เลิกใช้แล้วในปี 2022 — modern browser มี built-in แทน)

#### Productivity (ที่ควรพิจารณาก่อนติดตั้ง)

- **Vimium** (sf) — keyboard navigation (open source)
- **Dark Reader** (sf) — dark mode สำหรับเว็บ (open source)
- **Tab Groups extensions** — ขึ้นกับ workflow

#### Developer Tools

- **React DevTools / Vue DevTools** — ของ vendor ตรง
- **JSON Viewer** — formatting JSON
- **Wappalyzer** — ดู tech stack ของเว็บ

### Extensions ที่ควรหลีกเลี่ยง

#### กลุ่มที่อันตรายโดยทั่วไป

- ❌ **VPN extension ฟรี** ที่ไม่ใช่ของ VPN service ที่จ่ายเงิน — Hola, SuperVPN, etc.
- ❌ **YouTube downloader** — ส่วนใหญ่ฝัง malware
- ❌ **Coupon finder / shopping assistant** — เก็บทุก URL คุณเข้า ไปขายให้ ad network
- ❌ **PDF converter** ที่ไม่ใช่ของ Adobe / well-known
- ❌ **Screen recorder** ที่ขอ permission มากผิดปกติ

#### กลุ่มที่ควรเลิกใช้

- ⚠️ The Great Suspender (ถูก hijack ปี 2021)
- ⚠️ Web Developer (เคยถูก hijack)
- ⚠️ Edit This Cookie (เคยถูก hijack)

ตรวจสอบใน [extension-monitor.com](https://chrome-stats.com) หรือ [crxcavator.io](https://crxcavator.io) ก่อนติดตั้ง

### Tools ตรวจ Extension

- **CRXcavator** ([crxcavator.io](https://crxcavator.io)) — analyze Chrome extension หา risk
- **Chrome Stats** ([chrome-stats.com](https://chrome-stats.com)) — ดูสถิติ + history ของ extension
- **chrome://extensions/** — เปิดดู permission แต่ละตัว

---

## ส่วนที่ 2: HTTPS Everywhere — ดู Padlock ไม่พอ

### Padlock ไม่ได้แปลว่า "ปลอดภัย"

หลายคนคิดว่า:

> "เห็น 🔒 padlock = ปลอดภัย ส่งข้อมูลได้"

ถูก **ครึ่งหนึ่ง** ครับ

🔒 padlock แปลว่า:
- ✅ การเชื่อมต่อนี้เป็น **HTTPS** (encrypt ระหว่างทาง)
- ✅ Server มี **certificate ที่ valid** จาก CA ที่เชื่อถือ
- ✅ Domain ใน URL ตรงกับใน certificate

🔒 padlock **ไม่ได้** แปลว่า:
- ❌ เว็บนี้คือเว็บที่คุณคิดว่าเป็น
- ❌ ไม่ใช่เว็บปลอม
- ❌ ไม่ใช่ phishing
- ❌ Server ปลายทางไม่ได้ถูกแฮก

### กรณีที่ Padlock เขียวแต่เป็นเว็บปลอม

ในปี 2024 — **80%+ ของเว็บ phishing** ใช้ HTTPS เพราะ:

1. Let's Encrypt ออก certificate **ฟรี** ในไม่กี่นาที
2. ผู้โจมตีจดทะเบียน domain ปลอม → ขอ cert → ได้ HTTPS

ตัวอย่าง:

```
✅ Real:    https://login.scb.co.th
❌ Fake:    https://login-scb.co.th   (มี padlock เขียวเหมือนกัน!)
```

ทั้งสองเว็บมี padlock — แต่ตัวที่สองคือ phishing site

### วิธีเช็ค Certificate อย่างละเอียด

#### 1. คลิกที่ padlock

- **Chrome / Edge:** คลิก padlock → "Connection is secure" → "Certificate is valid"
- **Firefox:** คลิก padlock → arrow → "More information"
- **Safari:** คลิก padlock → "Show Certificate"

#### 2. ตรวจ "Issued To" + "Issued By"

```
Issued To:
  Common Name (CN): scb.co.th         ← ตรวจตรงนี้
  Organization:    Siam Commercial Bank Public Company Limited

Issued By:
  Common Name (CN): GlobalSign RSA OV SSL CA 2018
  Organization:    GlobalSign nv-sa
```

ดู:
- **CN** ตรงกับเว็บที่คุณเข้าไหม?
- **Organization** เป็นบริษัทที่ถูกต้องไหม?
- **Issuer** เป็น CA ที่เชื่อถือได้ไหม? (Let's Encrypt, DigiCert, GlobalSign, etc.)

#### 3. EV Certificate (Extended Validation)

เก่าๆ บางเว็บมี **EV cert** ที่ตอนคลิก padlock จะเห็นชื่อบริษัทเป็นสีเขียว — ปัจจุบัน browser ส่วนใหญ่ลบฟีเจอร์นี้แล้วเพราะ user ไม่ได้สนใจ ทำให้ EV cert ไม่ค่อยมีประโยชน์

### Mixed Content Warning

เว็บที่เป็น HTTPS แต่ load **resource ที่เป็น HTTP** — browser จะแจ้ง "Mixed content" + ปิด padlock

ตัวอย่าง:
```html
<!-- HTML ส่งผ่าน HTTPS -->
<img src="http://example.com/logo.png">  <!-- ← HTTP! -->
```

→ ผู้โจมตีอาจแก้ไขรูปนั้นได้ → ในกรณีของ JavaScript = อันตรายมาก

### HTTPS Everywhere Built-in

Modern browsers ในปี 2026 มี **HTTPS-First Mode** built-in:

- **Chrome:** Settings → Privacy and security → Security → "Always use secure connections"
- **Firefox:** Settings → Privacy & Security → "HTTPS-Only Mode" → "Enable in all windows"
- **Safari:** Settings → Privacy → "Use Secure Connections" (Safari 17+)
- **Edge:** Settings → Privacy → "Automatic HTTPS" → "Switch to HTTPS"

→ **เปิดเลย** browser จะ upgrade HTTP → HTTPS อัตโนมัติทุกครั้งที่ทำได้

### Certificate Transparency Logs

ทุก certificate ที่ออกโดย CA หลักจะถูก log ลง **Certificate Transparency** logs สาธารณะ — ใครก็ตรวจสอบได้:

- [crt.sh](https://crt.sh) — ค้น certificate ตาม domain
- ใช้ตรวจว่ามีใครออก cert ในชื่อ domain คุณโดยไม่ได้รับอนุญาตไหม

ตัวอย่าง: เข้า [crt.sh/?q=scb.co.th](https://crt.sh/?q=scb.co.th) → เห็นทุก cert ที่ออกในชื่อ scb.co.th

---

## ส่วนที่ 3: Cookie Consent ≠ Cookie Safety

### Cookie คืออะไรกันแน่

**Cookie** คือไฟล์เล็กๆ ที่เว็บ store ใน browser ของคุณ เพื่อ:

- จำว่าคุณ login (session cookie)
- จำการตั้งค่า (language, theme)
- ติดตามพฤติกรรม (analytics, ads)

ฟังดู ไม่อันตราย ใช่ไหม? แต่...

### First-Party vs Third-Party Cookies

#### First-Party

Cookie ที่ **เว็บที่คุณเปิด** เก็บไว้ — ปกติจำเป็น (login session, preferences)

ตัวอย่าง: เปิด `facebook.com` → Facebook เก็บ cookie เพื่อจำว่าคุณ login

#### Third-Party

Cookie ที่ **เว็บอื่น** (ที่ไม่ใช่ที่คุณเปิด) เก็บไว้ — ส่วนใหญ่เป็น **tracker**

ตัวอย่าง: เปิด `news.com` → ใน news.com มี ad ของ Google/Meta → Google/Meta ฝัง cookie ในเครื่องคุณผ่าน news.com

ผลคือ Google/Meta รู้ว่าคุณเข้าเว็บไหนบ้าง — ทุกเว็บที่มี ad ของเขา (ซึ่งคือ 90% ของอินเทอร์เน็ต)

### Cookie Consent Banner — มันแก้อะไร?

ตั้งแต่ **GDPR** (EU, 2018) และ **PDPA** (ไทย, 2022) ทุกเว็บต้องโชว์ **cookie consent banner**:

```
┌─────────────────────────────────────────────┐
│ This website uses cookies to improve your   │
│ experience.                                 │
│                                             │
│  [Accept All]  [Reject All]  [Manage]      │
└─────────────────────────────────────────────┘
```

ปัญหาคือ:

1. **"Accept All" สีเด่น** — "Reject All" สีจาง → user กด accept ตามนิสัย
2. **"Manage" ซับซ้อน** — มี 200+ vendor ที่ต้อง toggle ทีละตัว → user ยอมแพ้
3. **"Legitimate interest"** — โหมดพิเศษที่ไม่ต้องขอ consent → vendor ใช้ตรงนี้ทุกครั้ง

ผลคือ — banner ออกมาเพื่อ "compliance" แต่ practical คือ **tracker ก็ยังตามคุณอยู่**

### กลยุทธ์จัดการ Cookie ที่ใช้ได้จริง

#### 1. Block Third-Party Cookies By Default

- **Chrome:** Settings → Privacy and security → Cookies → "Block third-party cookies"
- **Firefox:** ใช้ "Strict" tracking protection (default)
- **Safari:** "Prevent cross-site tracking" (default)
- **Edge:** Settings → Privacy → "Strict tracking prevention"

→ จะลด tracker ได้ ~80%

#### 2. ใช้ uBlock Origin

uBlock Origin block:
- Ads
- Third-party trackers
- Malicious sites
- Cookie consent banners (ผ่าน "Annoyances" filter)

#### 3. Browser ที่ Privacy-First

- **Brave** — Chromium-based แต่บล็อก ads + tracker by default
- **Firefox + tracking protection strict**
- **Safari** — Apple บล็อก tracker เยอะที่สุดใน mainstream browsers

#### 4. Container Tabs (Firefox)

**Multi-Account Containers** ใน Firefox ทำให้แต่ละ tab อยู่ใน "container" ต่างกัน — cookies ไม่ปนข้ามกัน:

- Tab 1 (Personal): cookies ของบัญชีส่วนตัว
- Tab 2 (Work): cookies ของบัญชีงาน
- Tab 3 (Shopping): cookies ของบัญชี shopping

→ Facebook ใน Tab 1 มองไม่เห็น tab อื่น

#### 5. Clear Cookies เป็นระยะ

ตั้ง browser ให้ clear cookies ทุกครั้งปิด:

- **Chrome:** Settings → Privacy → Cookies → "Clear cookies when you close all windows"
- **Firefox:** Settings → Privacy → "Delete cookies when Firefox is closed"

(ใส่ exception เว็บที่อยากให้จำ login ไว้)

### Tracking ที่ Cookie ไม่ใช่อย่างเดียว

แม้ปิด cookie หมด — ผู้โจมตียังตามคุณได้ผ่าน:

#### 1. Browser Fingerprinting

รวมข้อมูลของ browser เพื่อสร้าง "fingerprint":

- User Agent string
- Screen resolution
- Installed fonts
- Time zone
- Language
- Hardware (GPU info, etc.)

ผลคือ — แม้คุณ clear cookie หมด เปิด incognito tab — เว็บก็ยังจำได้ผ่าน fingerprint

ทดสอบ: [coveryourtracks.eff.org](https://coveryourtracks.eff.org) — ดูว่า browser ของคุณ unique แค่ไหน

#### 2. Web Storage (localStorage)

แทน cookie ก็ได้ — เก็บใน `localStorage`/`sessionStorage` ของ browser → ไม่ได้ลบเมื่อ clear cookie

#### 3. ETag / Cache

HTTP cache header ทำให้ tracker เก็บ ID ใน cache ได้

#### 4. IP Address

แม้ปิดทุกอย่าง — IP ยังเปิดเผย ISP, ตำแหน่งทั่วไป

→ **ใช้ VPN** เพื่อปิด IP

---

## ส่วนที่ 4: Browser Permissions — Allow โดยไม่อ่าน = อันตราย

### Permissions ที่ทุกเว็บขอ

ทุกครั้งคุณเปิดเว็บใหม่ ที่ปุ่ม "Allow":

- **Notifications** — เว็บส่ง notification ขึ้นมาบนหน้าจอ
- **Location** — เว็บเห็น GPS ของคุณ
- **Camera** — เว็บเปิดกล้องได้
- **Microphone** — เว็บฟังเสียงได้
- **Files** — เว็บอ่าน/เขียนไฟล์ได้
- **Clipboard** — เว็บอ่าน clipboard ของคุณ
- **MIDI / Bluetooth** — เว็บเชื่อมต่ออุปกรณ์
- **Background sync** — เว็บทำงานต่อแม้ปิด tab

### ปัญหาคือ User กด "Allow" ทุกครั้งโดยไม่อ่าน

นี่คือพฤติกรรมปกติของมนุษย์ — เห็นป๊อปอัพ → กดเร็วๆ ให้มันหายไป → ไม่อ่าน

ผลคือ — เว็บ random ที่คุณเปิดเมื่อปีก่อน อาจยังมี permission อยู่ในเครื่องคุณตอนนี้

### Audit Permission ของคุณตอนนี้

#### Chrome / Edge

```
Settings → Privacy and security → Site Settings → 
   Notifications / Location / Camera / Microphone (เลือกทีละอัน)
```

ดู list เว็บที่มี permission → **ลบทุกตัวที่ไม่จำเป็น**

#### Firefox

```
Settings → Privacy & Security → Permissions → 
   Settings ของแต่ละชนิด
```

#### Safari

```
Settings → Websites → 
   Notifications / Location / Camera / Microphone
```

### กลยุทธ์ Permission ที่แนะนำ

#### Notification

- **Default:** Block ทั้งหมด
- **Allow เฉพาะ:** Slack, Gmail, Calendar (ที่ใช้จริง)

#### Location

- **Default:** Block
- **Allow:** Google Maps (ใช้จริง), บริการขนส่ง

#### Camera / Microphone

- **Default:** Block
- **Allow:** Google Meet, Zoom, MS Teams (ใช้จริงเท่านั้น)

#### Clipboard

- **Default:** Block ในทุกกรณี (ยกเว้นเว็บที่ใช้ paste มากๆ)

### "Allow Once" vs "Allow Always"

ทุกครั้งที่เว็บขอ permission — เลือก "Allow once" ก่อน → ถ้าใช้บ่อยจริงๆ ค่อยเปลี่ยนเป็น "Allow always"

---

## ส่วนที่ 5: Email Identity Management

### ปัญหา: Email เดียวสำหรับทุกที่

หลายคนใช้ email หลัก (เช่น `tanin@gmail.com`) เป็น login สำหรับ:

- Online shopping (Lazada, Shopee, Amazon)
- Streaming (Netflix, Spotify)
- News / blog (signup เพื่ออ่านบทความ)
- Forum / community
- Newsletter
- Unknown sketchy services ที่เคยลอง

ผลคือ:

1. **เมื่อเว็บใดเว็บหนึ่งถูกแฮก** → email ของคุณรั่ว
2. **ผู้โจมตีรู้ว่าคุณ "อยู่"** ที่ไหนบ้าง → spear phishing ได้
3. **Spam** ทะลักเข้า inbox
4. **Marketing** ขายต่อ list ของคุณ → spam หนักขึ้น

### Solution: Email Aliases / Disposable Emails

แนวคิด — ใช้ **email ต่างกัน** สำหรับแต่ละบริการ → ติดตามได้ว่าใครรั่ว → block ได้เป็นรายบริการ

#### ระดับที่ 1: Plus Addressing (Gmail, Outlook)

Gmail / Outlook อนุญาตให้คุณใส่ `+anything` ก่อน `@`:

```
tanin@gmail.com         ← email หลัก
tanin+netflix@gmail.com ← ใช้ signup Netflix
tanin+lazada@gmail.com  ← ใช้ signup Lazada
tanin+shopee@gmail.com  ← ใช้ signup Shopee
```

email เหล่านี้ทั้งหมดเข้า inbox เดียวกัน — **แต่** คุณรู้ว่าใครส่งมา และ filter ได้

**ข้อจำกัด:** ผู้โจมตีถอด `+xxx` ออกได้ → กลับมาเป็น email หลัก → ไม่ใช่ความลับสมบูรณ์

#### ระดับที่ 2: Custom Domain + Catch-all

ถ้าคุณมี domain ของตัวเอง (เช่น `tanin.dev`) — ตั้ง catch-all email:

```
netflix@tanin.dev
lazada@tanin.dev
random-site-2024@tanin.dev
```

ทุก email ที่ส่งมาที่ `*@tanin.dev` เข้าตู้เดียว — แต่:
- ไม่มี pattern ให้ถอดออก
- ลบ alias ใดได้ทันที

#### ระดับที่ 3: Email Forwarding Service

บริการที่สร้าง alias ให้คุณ:

- **[Apple Hide My Email](https://support.apple.com/guide/icloud/use-hide-my-email-mma37cd28e0c/icloud)** — ฟรี (สำหรับ iCloud+)
- **[SimpleLogin](https://simplelogin.io)** (open source) — alias ไม่จำกัด
- **[Firefox Relay](https://relay.firefox.com)** — โดย Mozilla
- **[AnonAddy / addy.io](https://addy.io)** — open source
- **DuckDuckGo Email Protection** — ฟรี

วิธีใช้:

1. สมัครบริการ
2. ตอน signup เว็บ → กด button extension "Generate Alias"
3. ได้ email สุ่ม เช่น `random123@simplelogin.io`
4. Email ที่ส่งมาที่นี่ → forward ไป email หลักของคุณ
5. ถ้าโดน spam — ปิด alias นั้นเฉพาะตัว

#### ระดับที่ 4: Disposable Email สำหรับ One-time Use

- **[10minutemail.com](https://10minutemail.com)** — email ที่หายเองใน 10 นาที
- **[temp-mail.org](https://temp-mail.org)** — ใกล้เคียง
- **[guerrillamail.com](https://guerrillamail.com)** — disposable email

ใช้สำหรับ — signup เว็บที่ใช้ครั้งเดียว ไม่มีเจตนาใช้ต่อ

### กลยุทธ์ Email ของผม (แนะนำ)

ผมแบ่ง email เป็น 4 tier:

#### Tier 1: Primary (ปกป้องสุดชีวิต)

- 1 email สำหรับเรื่องสำคัญที่สุด: ธนาคาร, ภาษี, ราชการ, ประกัน
- ห้าม share ห้ามใช้ signup อะไรอื่น
- 2FA + hardware key

#### Tier 2: Work / Professional

- Email ของบริษัท + 1 personal email สำหรับ professional life
- ใช้สมัคร LinkedIn, GitHub, Stack Overflow, conference
- 2FA เปิด

#### Tier 3: Daily Use (Email หลักทั่วไป)

- 1 Gmail / Outlook ส่วนตัวที่ใช้ทั่วไป — Netflix, Spotify, ฯลฯ
- เปิด plus addressing
- 2FA เปิด

#### Tier 4: Throwaway / Aliased

- ใช้ SimpleLogin / Hide My Email สำหรับ:
  - Newsletter
  - Forum
  - One-time signup
  - Sketchy services ที่ไม่แน่ใจ

→ ถ้า Tier 4 รั่ว = ไม่กระทบอะไร  
→ ถ้า Tier 1 รั่ว = หายนะ

---

## ส่วนที่ 6: Browser Hardening Checklist

รวมทุกอย่างที่พูดมาเป็น checklist ที่ทำได้ใน 30 นาที:

### Step 1: Choose Browser

แนะนำ:
- **Brave** หรือ **Firefox + tracking protection strict** สำหรับ privacy
- **Chrome / Edge** ถ้าต้องใช้กับ work account (แต่ harden ตามด้านล่าง)
- **Safari** บน Apple devices ดี privacy โดย default

### Step 2: Settings ที่ต้องเปลี่ยน

- [ ] **HTTPS-Only Mode** เปิด
- [ ] **Block third-party cookies** เปิด
- [ ] **Tracking protection: Strict** (Firefox) / "Enhanced" (Chrome)
- [ ] **Send "Do Not Track"** เปิด (มาช่วยน้อย แต่ไม่เสียอะไร)
- [ ] **Phishing & Malware Protection** เปิด (default)
- [ ] **Safe Browsing: Enhanced** (Chrome) / "Maximum Protection"
- [ ] **Block notifications by default**
- [ ] **Block location by default**
- [ ] **Block camera/microphone by default**

### Step 3: ติดตั้ง Extensions ที่จำเป็น (น้อยที่สุด)

- [ ] **uBlock Origin** (ถ้ายังไม่ได้ใช้ Brave / Safari)
- [ ] **Password manager** (1Password / Bitwarden)
- [ ] **Email forwarding** (SimpleLogin / Hide My Email)

### Step 4: Audit Existing Permissions

- [ ] ลบ notification permission ของเว็บที่ไม่จำเป็น
- [ ] ลบ location permission ของเว็บที่ไม่จำเป็น
- [ ] ลบ camera/mic permission ของเว็บที่ไม่จำเป็น

### Step 5: Audit Existing Extensions

- [ ] ลบ extension ที่ไม่ได้ใช้ใน 1 เดือน
- [ ] ตรวจ permission ของแต่ละ extension
- [ ] ตรวจ developer ของแต่ละ extension

### Step 6: Setup Email Strategy

- [ ] เลือก primary email สำหรับ tier 1
- [ ] เปิด 2FA + hardware key
- [ ] สมัคร SimpleLogin / Hide My Email
- [ ] ลิสต์ services ทั้งหมด → migrate ไป alias ภายใน 1 เดือน

### Step 7: Test Yourself

- [ ] [coveryourtracks.eff.org](https://coveryourtracks.eff.org) — fingerprint test
- [ ] [haveibeenpwned.com](https://haveibeenpwned.com) — เช็ค email หลักรั่วหรือยัง
- [ ] [browserleaks.com](https://browserleaks.com) — เช็ค leak ต่างๆ

---

## Action Items

### วันนี้

- [ ] เปิด **HTTPS-Only Mode** ใน browser
- [ ] เปิด **Strict tracking protection**
- [ ] Block third-party cookies
- [ ] ติดตั้ง **uBlock Origin** (ถ้ายังไม่มี)

### สัปดาห์นี้

- [ ] Audit browser extensions — ลบที่ไม่ใช้
- [ ] Audit browser permissions — revoke ที่ไม่จำเป็น
- [ ] เลือก email forwarding service สมัคร
- [ ] เริ่มใช้ alias สำหรับ signup ครั้งใหม่

### เดือนนี้

- [ ] Migrate email signup เก่าๆ ไปใช้ alias ทีละบริการ
- [ ] แยก browser profile (work / personal)
- [ ] ติดตั้ง **Multi-Account Containers** (Firefox) — ถ้าใช้ Firefox

### สำหรับ Tech Lead

- [ ] **Allowlist extensions** ผ่าน MDM ในเครื่องบริษัท
- [ ] **Block third-party cookies** ผ่าน group policy
- [ ] **Force HTTPS-Only Mode**
- [ ] **Train ทีม** เรื่อง phishing extension
- [ ] **Subscribe phishing alert feeds** + post ใน team channel

---

## บทเรียนชีวิตจากบทความนี้

> **ทุกครั้งที่กด "Allow" หรือ "Accept" ให้ถามตัวเองว่ากำลังแลกอะไรอยู่**

ในโลกออนไลน์ ทุกอย่างที่ "ฟรี" จริงๆ มี **ราคาที่คุณจ่ายเป็น data**:

- Facebook ฟรี → คุณจ่ายด้วย attention + data
- Gmail ฟรี → Google scan email เพื่อ AI / ad
- Extension ฟรี → ผู้พัฒนาเก็บ data ขายต่อ
- VPN ฟรี → ขายข้อมูลของคุณ
- Quiz online ฟรี → เก็บข้อมูลพฤติกรรม

ไม่ได้แปลว่า "ทุกอย่างฟรีคืออันตราย" — แต่ต้อง **รู้ว่ากำลังแลกอะไร**

เปรียบเทียบกับชีวิตจริง — ทุกครั้งที่:

- เซ็น contract → คุณกำลังให้สิทธิ์อะไร?
- ตอบ "ใช่" ในสิ่งที่ขออยู่ → ภาระอะไรตามมา?
- กดยอม T&C 50 หน้า → คุณรู้ไหมว่ามีอะไรในนั้น?
- โอนเงินให้ใคร → ปลอดภัยจริงไหม?

> **การ "say yes" โดยไม่อ่าน** เป็นนิสัยอันตราย ทั้งใน browser และในชีวิต

ทักษะของคนที่ป้องกันตัวเองได้ คือ:

1. **Pause** — หยุดก่อนกด
2. **Read** — อ่านสิ่งที่ขอ
3. **Question** — ถามว่าจำเป็นจริงไหม
4. **Verify** — ตรวจสอบจากแหล่งอื่น
5. **Choose** — กดตามที่ตัดสินใจแล้ว ไม่ใช่ตามนิสัย

ในชีวิตจริง คนที่:
- กดเซ็นทุก contract ที่ส่งมา → โดนผู้รับเหมาเอาเปรียบ
- ตอบ "ได้" ทุกคำขอจากเพื่อนร่วมงาน → burnout
- ยอมรับเงื่อนไขทุกอย่างจาก partner → ความสัมพันธ์ที่ไม่สมดุล
- ลงทุนตามคำชวน "ได้กำไรแน่นอน" → หมดตัว

> **คนที่ "no" ไม่เป็น มักจะแพ้คนที่ "yes" เลือก**

ในโลกที่ทุกคนพยายาม "ขอ" จากคุณ — สิ่งสำคัญที่สุดคือทักษะการ "select" ว่าจะให้อะไรกับใคร และเมื่อไหร่

เริ่มจาก browser ของคุณวันนี้ — ขยายไปทั้งชีวิต

---

## อภิธานศัพท์ (Glossary)

- **Browser Extension:** plugin ที่เพิ่มฟีเจอร์ให้ browser
- **First-Party Cookie:** cookie ที่ domain ที่คุณเปิดเก็บไว้
- **Third-Party Cookie:** cookie ที่ domain อื่นเก็บไว้ (ส่วนใหญ่เป็น tracker)
- **Tracking Protection:** ฟีเจอร์ของ browser ที่บล็อก tracker
- **Browser Fingerprinting:** การ track user ผ่านลักษณะของ browser ไม่ใช่ cookie
- **Mixed Content:** หน้าเว็บ HTTPS ที่ load resource HTTP
- **EV Certificate (Extended Validation):** SSL cert ที่ verify ตัวตนของ org
- **Certificate Transparency:** log สาธารณะของ SSL cert ทุกใบที่ออก
- **HTTPS Everywhere:** เก่าใช้ extension ปัจจุบันใช้ HTTPS-Only Mode built-in
- **Email Alias:** email สมมติที่ forward ไป email หลัก
- **Plus Addressing:** Gmail/Outlook feature เพิ่ม `+xxx` ก่อน @
- **Disposable Email:** email ใช้ครั้งเดียวที่หายเอง
- **uBlock Origin:** ad/tracker blocker open source ที่แนะนำ
- **CRXcavator:** เครื่องมือวิเคราะห์ Chrome extension
- **Container Tabs:** ฟีเจอร์ Firefox แยก context ของแต่ละ tab
- **Permission Request:** popup ที่เว็บขอสิทธิ์ใช้ feature ของ browser
- **GDPR:** กฎหมาย privacy ของ EU
- **PDPA:** กฎหมาย privacy ของไทย

---

## สรุป

1. **Browser extension เป็น attack surface ที่ใหญ่** — ใช้น้อยที่สุด ตรวจสอบเป็นระยะ
2. **HTTPS padlock ไม่ได้แปลว่าปลอดภัย** — phishing ก็มี HTTPS ตรวจ certificate ให้ละเอียด
3. **Cookie consent ≠ cookie safety** — ตั้ง browser block third-party cookies + ใช้ uBlock
4. **Browser permissions** ที่กด "Allow" ไปแล้วยังอยู่ — audit เป็นระยะ
5. **Email aliases** = layer ป้องกัน identity ที่ทรงพลัง — ใช้ SimpleLogin / Hide My Email
6. **Browser fingerprinting** เป็นปัญหานอกเหนือ cookie — ใช้ Brave/Firefox + uBlock ช่วย
7. **Online hygiene** เริ่มที่นิสัย "หยุดก่อนกด" — เป็นทักษะชีวิตที่ใช้ได้ทุกที่

ในบทถัดไป เราจะเข้าเรื่อง **Ransomware** — malware ที่ encrypt ข้อมูลทั้งเครื่องคุณ แล้วเรียกค่าไถ่ — เป็นภัยอันดับหนึ่งของบริษัทในปัจจุบัน

— Claude Opus 4.6
