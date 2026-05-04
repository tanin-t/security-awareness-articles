# Review Guide — CyberSecurity Awareness Series

Checklist สำหรับ AI agent ที่จะ review บทความก่อน publish

ใช้คู่กับ [writing-guide.md](writing-guide.md)

---

## A. Audience & Flow

- [ ] บทความเข้าถึงได้ทั้ง **คนทั่วไป + dev** ไม่ใช่ dev เพียงอย่างเดียว
- [ ] มี story / case จริงเปิดบทความ (ไม่ใช่ทฤษฎีก่อน)
- [ ] เนื้อหาสำหรับคนทั่วไปอยู่ก่อน เนื้อหาเชิงเทคนิคสำหรับ dev อยู่หลัง
- [ ] ส่วน dev (ถ้ามี) เปิดด้วยประโยคที่บอกว่า "ถ้าไม่ใช่ dev จะข้ามได้" เพื่อให้คนทั่วไปไม่หลุดจากกลุ่มเป้าหมาย
- [ ] มี Action Items ที่ทำได้เลย — 4 ข้อสำหรับ user ทั่วไป + 4 ข้อเพิ่มสำหรับ dev (ถ้ามีส่วน dev)
- [ ] มี **"บทเรียนชีวิต"** ปิดท้าย — ใช้ได้นอกบริบท cybersecurity ด้วย

---

## B. Term Definitions

- [ ] **ไม่มี Glossary section ที่ท้ายบทความ**
- [ ] **ไม่มี footnote `[^N]`** สำหรับนิยาม
- [ ] ทุก technical term ที่ปรากฏครั้งแรก มีคำอธิบาย inline ในรูป `**term** (คำอธิบาย)` หรือ `**term** — คำอธิบาย`
- [ ] คำอธิบายสั้น ไม่เกิน 1-2 ประโยค ไม่ทำให้ flow บทความสะดุด
- [ ] ถ้ามี Thai equivalent ที่ดี ใช้คู่กับ English term

**วิธีเช็ค:** `grep -n '\[\^' article.md` ต้องไม่ return อะไร

---

## C. External Links

- [ ] **ทุก URL ที่อ้างอิง ต้อง fetch ด้วย WebFetch ก่อน เพื่อ verify ว่ายังใช้งานได้**
- [ ] ลิงก์ทุกตัวพาไปยังเนื้อหาที่ตรงกับสิ่งที่บทความอ้าง (ไม่ใช่หน้า home หรือ search result)
- [ ] หลีกเลี่ยงลิงก์ที่ระบุตัวบุคคลในแบบที่อาจ doxxing
- [ ] **เคยมีปัญหามาแล้วกับ:** Imperva blog (404), Ars Technica บางเส้นทาง — เลือกแหล่งอื่นเช่น TechCrunch, KrebsOnSecurity, Wikipedia แทน

**คำสั่ง verify:**

```bash
# extract ทุก URL และ check
grep -oE 'https?://[^)]+' article.md | sort -u
# จากนั้นใช้ WebFetch ทุกตัว
```

---

## D. Secret Scanner Safety

Push protection ถูกปิดแล้ว — สามารถใช้ตัวอย่างที่หน้าตาคล้ายของจริงได้ **แต่ต้อง mask ส่วนกลางด้วย `*`** เสมอ ห้ามใส่ token ที่ดูเหมือนใช้งานได้จริง

- [ ] ตัวอย่างที่หน้าตาคล้ายของจริง ต้องมี `***...***` mask ตรงกลาง เช่น `sk_live_4eC39HqL*****************szdp7dc`
- [ ] หรือใช้ `<placeholder>` syntax ก็ยังโอเค (`<your-stripe-secret-key>`, `STRIPE_SECRET_KEY_HERE`)
- [ ] **ห้ามใส่ token ที่อาจเป็น key จริง** (ไม่มี `*` mask ตรงกลาง) — เช็ค `grep` ด้านล่าง

**วิธีเช็ค:**

```bash
# หา token-like strings ที่ไม่มี * mask
grep -nE '(sk_live_|sk_test_|AKIA[A-Z0-9]{16}|ghp_|ghs_|sk-[A-Za-z0-9]{20,})' article.md \
  | grep -v '\*'
```

ต้องไม่ return อะไร (มีก็ได้แต่ต้องมี `*` mask)

---

## E. Code Examples

- [ ] Code มี comment อธิบายว่าทำไมถึงผิด/ถูก เมื่อยกตัวอย่างเปรียบเทียบ
- [ ] ใช้ภาษาที่ realistic กับ stack ของ dev ทั่วไป (Python / JavaScript / TypeScript)
- [ ] ไม่มี syntax error ใน code block (ลอง parse ด้วยตา หรือ run ถ้าเป็นไปได้)

---

## F. Tone & Language

- [ ] ภาษาไทย ผสม English technical terms (ไม่แปลทุกคำ)
- [ ] ลงท้ายด้วย "ครับ" บางจุด แต่ไม่ทุกประโยค
- [ ] มี analogy ในชีวิตจริงเสมอเมื่อเจอ concept ใหม่
- [ ] ไม่ใช้น้ำเสียงตำหนิ user (เช่น "คุณโง่ที่ใช้รหัสผ่านซ้ำ")
- [ ] ไม่ใช้ technical jargon โดยไม่มีคำอธิบาย

---

## G. Header / Metadata

- [ ] มี header ครบ: title, series tag, ผู้เขียน + บรรณาธิการ
- [ ] มี link "อ่านต่อ →" ไปบทความถัดไป (ถ้ามี)
- [ ] มี footer `*CyberSecurity Awareness Series — บทความที่ N จาก 23*`

---

## H. Cross-references

- [ ] Reference ไปบทความอื่นใช้รูปแบบ `→ อ่านเพิ่มในบทความที่ X`
- [ ] หมายเลขบทความตรงกับ outline ใน [readme.md](readme.md)
- [ ] ไม่อ้างอิงถึงบทความที่ยังไม่มีจริงโดยไม่ระบุว่ายังเขียนไม่เสร็จ

---

## I. Action Items

- [ ] Action item ขึ้นต้นด้วยกริยา ("เลิก...", "ตรวจ...", "ตั้ง...")
- [ ] ทำได้วันนี้ ไม่ใช่ "ในระยะยาว"
- [ ] มี link ที่ไปทำได้ทันที (เครื่องมือที่ใช้ฟรีได้)
- [ ] เลขต่อกันระหว่าง section user (1-4) และ section dev (5-8)

---

## J. Final Pass

- [ ] อ่านบทความจากต้นจนจบเหมือนคนอ่านครั้งแรก ดูว่า flow โอเคไหม
- [ ] ไม่มีศัพท์ที่อ่านแล้วต้อง google ทันทีโดยไม่มีบริบท
- [ ] บทเรียนชีวิตท้ายบทความ ใช้ได้กับสถานการณ์ทั่วไป ไม่ใช่แค่ context cybersecurity
- [ ] ความยาวเหมาะสม — ไม่ยาวจนเหนื่อย ไม่สั้นจนไม่ครอบคลุม

---

## วิธีใช้ checklist นี้

1. Run `grep` commands ใน section B, C, D เพื่อ catch issue เชิงกลก่อน
2. ใช้ WebFetch ตรวจทุก URL ใน section C
3. อ่านบทความ 1 รอบเต็ม ติ๊กแต่ละ checkbox
4. ส่งผลกลับให้ editor พร้อมประเด็นที่ต้องแก้ (ถ้ามี)
