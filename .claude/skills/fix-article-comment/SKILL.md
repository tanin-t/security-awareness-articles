---
name: fix-article-comment
description: Apply GitHub PR review comments to articles in this CyberSecurity Awareness Series, then (after approval) distill lessons into writing-guide.md and review-guide.md. Invoke when the user references PR review comments or asks to update an article based on feedback.
---

# /fix-article-comment

Workflow สำหรับใช้ PR review comments เป็น input ในการแก้บทความ และเก็บบทเรียนกลับเข้า writing-guide / review-guide

---

## Args

- `<PR-number>` — เลข PR (ถ้าไม่ระบุ ให้ดึงจาก current branch ด้วย `gh pr view --json number`)
- `--finalize` — โหมด harvest บทเรียนหลัง PR approved (ดู Phase 2)

---

## Phase 1 — Apply review comments

### 1. ดึง comments ทั้งหมดจาก PR

ใช้ `gh` CLI สำหรับ read-only ops (auth ใช้งานได้ปกติสำหรับ read; เฉพาะ `gh pr create` เท่านั้นที่ติดปัญหา 401 ใน repo นี้):

```bash
PR=<number>

# General PR comments (issue-style)
gh api "repos/{owner}/{repo}/issues/$PR/comments" --jq '.[] | {user: .user.login, body: .body, created_at: .created_at, url: .html_url}'

# Inline review comments (file-anchored)
gh api "repos/{owner}/{repo}/pulls/$PR/comments" --jq '.[] | {user: .user.login, path: .path, line: .line, body: .body, created_at: .created_at, url: .html_url}'

# PR description + review summaries
gh pr view $PR --json title,body,reviews,files
```

แทน `{owner}/{repo}` ด้วย owner/repo จริง — ดึงจาก `gh repo view --json nameWithOwner` ก่อน

### 2. จัดกลุ่ม comment ตามประเภท

อ่าน comment ทั้งหมด แล้วจัดกลุ่มเป็น:

- **Content fix** — ผิดข้อมูล, ตัวอย่างไม่ตรง, อธิบายไม่ชัด
- **Link issue** — ลิงก์ 404, ลิงก์ไม่ตรงเนื้อหา
- **Term/definition** — ศัพท์ที่ยังไม่ได้อธิบาย, อธิบายแล้วยังไม่เข้าใจ
- **Tone/audience** — ตำหนิ user, technical เกินไปสำหรับคนทั่วไป
- **Structure** — section ขาด, ลำดับไม่ดี
- **Code/example** — code ผิด, ตัวอย่าง secret ที่อันตราย
- **Cross-reference** — link ไปบทความอื่นไม่ถูก
- **Other** — ที่ไม่เข้ากลุ่มข้างต้น

ใช้ TodoWrite สร้าง todo 1 ข้อต่อ comment เพื่อ track ว่าทำครบ

### 3. แก้บทความตาม comment

สำหรับแต่ละ comment:

- ถ้าเป็น inline comment (มี `path` + `line`): เปิดไฟล์นั้นที่ line นั้น แก้ตามที่ reviewer ขอ
- ถ้าเป็น general comment: หา section ที่เกี่ยวข้องในบทความ (อาจถามผู้ใช้ถ้าไม่ชัด)
- **เคารพเจตนา reviewer** — ถ้าไม่แน่ใจว่าเข้าใจถูก ถามผู้ใช้ก่อนแก้ ดีกว่าแก้ผิด
- ทุก link ที่เพิ่มหรือเปลี่ยน ต้อง verify ด้วย WebFetch ก่อน
- ทุกการเปลี่ยน secret-shaped string ต้องดูว่ามี `*` mask ตรงกลาง

### 4. Reply กลับใน thread

หลังแก้แต่ละ comment ให้ reply ใน thread นั้น (ใช้ `gh api` POST):

```bash
gh api "repos/{owner}/{repo}/pulls/$PR/comments/$COMMENT_ID/replies" \
  -X POST \
  -f body="แก้ตามคำแนะนำแล้วครับ — <สั้นๆ ว่าแก้อะไร> (commit: $SHA)"
```

หรือถ้าเป็น issue-style comment ใช้:

```bash
gh api "repos/{owner}/{repo}/issues/$PR/comments" -X POST -f body="..."
```

### 5. Commit และ push

```bash
git add articles/<file>.md
git commit -m "fix: address review comments on <article>"
git push
```

ห้ามใช้ `--no-verify` หรือ `--amend` ครับ — สร้าง commit ใหม่เสมอ

---

## Phase 2 — Finalize: harvest lessons into guides

ทำเมื่อ PR ถูก approve / merged แล้ว (`gh pr view $PR --json state,reviews`)

### 1. รวบรวม review feedback ทั้งหมด

อ่าน comment + diff ตั้งแต่ commit แรกของ branch จนถึง merge:

```bash
gh pr view $PR --json commits,reviews,comments
git log --oneline $BASE..$HEAD
```

### 2. สรุปบทเรียน

สำหรับแต่ละกลุ่ม comment ใน Phase 1 step 2 ถามตัวเอง:

- **กฎที่มีอยู่ใน writing-guide / review-guide ครอบคลุมเรื่องนี้แล้วหรือยัง?**
  - ครอบคลุมแล้ว แต่บทความนี้พลาด → ไม่ต้องแก้ guide แต่ flag ให้ระวังครั้งหน้า
  - ครอบคลุมแล้ว แต่กฎไม่ชัดพอ → แก้ wording ให้ชัดขึ้น
  - ยังไม่มี → เพิ่มกฎใหม่
- **เป็นบทเรียนระดับโครงสร้าง หรือแค่กรณีเฉพาะ?**
  - โครงสร้าง (เช่น "ห้าม assume reader เป็น dev") → เข้า writing-guide
  - กรณีเฉพาะ (เช่น "Imperva 404 มาแล้ว") → เข้า review-guide เป็น "เคยเจอมาแล้ว"

### 3. เสนอ update ก่อนแก้

แสดง diff ที่จะ apply ใน writing-guide / review-guide ให้ user confirm ก่อน — ห้าม commit เปลี่ยน guide โดย user ไม่อนุมัติ:

```
ผมจะเสนอเพิ่มกฎ X ใน writing-guide.md section Y เพราะใน PR #N reviewer ทักว่า ...
diff:
- (เก่า)
+ (ใหม่)

ตกลงไหมครับ?
```

### 4. Apply และ commit

หลัง user ตกลงแล้ว แก้ guide แล้ว commit:

```bash
git add writing-guide.md review-guide.md
git commit -m "guide: lesson from PR #$PR — <สั้นๆ ว่าเพิ่ม/แก้กฎอะไร>"
git push
```

### 5. (Optional) บันทึก memory

ถ้าบทเรียนเป็น pattern ที่จะเจอซ้ำ (เช่น แหล่งข่าวแหล่งใหม่ที่นิ่ง / ไม่นิ่ง) บันทึกเป็น project memory ด้วย เพื่อให้ session อนาคต recall ได้

---

## ข้อควรระวัง

- **`gh pr create` มีปัญหา 401 ใน repo นี้** — ถ้าต้องสร้าง PR ใหม่ ใช้ `curl` กับ `Authorization: Bearer $(gh auth token)` แทน (ดู memory `feedback_gh_cli_pr_create.md`) — แต่ skill นี้ไม่ create PR, แค่ comment + commit
- **อย่าตอบกลับ comment ที่เป็นคำถาม โดยไม่มีคำตอบจริง** — ถ้า reviewer ถามอะไรที่ตอบไม่ได้ ขอความเห็น user ก่อน
- **อย่า resolve thread เอง** — ปล่อย reviewer เป็นคน resolve เองเมื่อพอใจ
- **ทุกการ push เป็น public action** — ถ้าไม่แน่ใจว่าควร push ตอนนี้ ถาม user ก่อน
