# บทความที่ 11: Backup & Disaster Recovery — แผนสำรองที่ทุกคนต้องมี

**CyberSecurity Awareness Series — Part 4: Everyday Threats**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของบริษัทที่ "หายไปในคืนเดียว"

เดือนกุมภาพันธ์ ปี 2017 บริษัท web hosting ของรัสเซียชื่อ **GitLab** (เคยขายให้ wp-engine ก่อนเปลี่ยนชื่อ) — ไม่ใช่ครับ ผมหมายถึง **GitLab.com** เอง — มีเรื่องที่กลายเป็นตำนานในวงการ DevOps

วันที่ 31 มกราคม 2017 ตอน ~17:20 UTC พนักงาน DBA คนหนึ่งของ GitLab ทำงานเรื่อง maintenance database ในเวลาทำงาน — ขณะที่กำลังจะ clean up data จาก secondary database server เพราะ replication มีปัญหา เขาต้อง:

```bash
# คำสั่งที่ตั้งใจ
rm -rf /var/opt/gitlab/postgresql/data  # ลบเฉพาะ secondary server
```

แต่เขา SSH ผิด server โดยไม่รู้ตัว — กดคำสั่งบน **primary database server**

ภายในเสี้ยววินาที — **300GB ของ production database ถูกลบไป**

ทีม panic ไปดู backup:

| Backup Type | Status |
|---|---|
| **pg_dump** ทุก 24 ชั่วโมง | ❌ Empty file (script silently failed) |
| **Disk snapshot** | ❌ ไม่ได้ enable บน server นี้ |
| **Replication ไป secondary** | ❌ Broken มา 2 อาทิตย์แล้ว |
| **WAL archives** (point-in-time recovery) | ⚠️ ต้องลอง |
| **Manual backup ของ DBA** | ✅ มี! 6 ชั่วโมงก่อน |

สุดท้าย GitLab **กู้ได้** จาก manual backup ของ DBA ที่บังเอิญทำไว้ — แต่:

- **ข้อมูล 6 ชั่วโมง** หายไป — รวม **5,037 projects, 5,000 users, 707 issues**
- Public outage นาน **18 ชั่วโมง**
- GitLab live-streamed การ recovery บน YouTube — แสดงให้เห็นทุกขั้นตอน (transparent ที่น่าชื่นชม)

ที่น่าตกใจที่สุด — GitLab มี **5 backup mechanisms** แต่ใช้ไม่ได้ **4 ตัว** ในวันที่ต้องใช้จริง

[อ่าน post-mortem ของ GitLab เอง](https://about.gitlab.com/blog/2017/02/01/gitlab-dot-com-database-incident/)

> **"Backup ที่ไม่เคยทดสอบ — ไม่ใช่ backup"**

นี่คือบทเรียนที่ทุกคนต้องจำ และเป็นหัวใจของบทความนี้

---

## ทำไม Backup ไม่ใช่แค่เรื่อง Ransomware

หลายคนคิดว่า backup สำคัญแค่ตอนโดน ransomware (จากบทที่ 10) — **ผิดครับ**

ข้อมูลของคุณอาจหายได้จากเหตุผลมากมาย:

### 1. Hardware Failure

- **HDD พัง** — เฉลี่ย 2-5% per year (Backblaze stats) → 10 ปีโอกาสพัง 30%+
- **SSD ก็พังได้** — มี wear cycle ที่จำกัด
- **RAID ไม่ใช่ backup** — RAID ป้องกัน disk failure แต่ไม่ป้องกัน fire, theft, deletion
- **Phone หาย / ตกน้ำ / พัง** — รูป ทุกอย่างใน gallery หายเลย

### 2. Human Error

- ลบไฟล์ผิด
- Format disk ผิด
- `rm -rf /` (จริงๆ มี dev คน)
- Reset settings โดยไม่ตั้งใจ
- ส่งเอกสารทับฉบับเก่า

จาก stat ของ World Backup Day:
- **30% ของ user ไม่เคย backup**
- **113 phones** หายทุกนาทีทั่วโลก

### 3. Ransomware

ที่กล่าวในบทก่อน

### 4. Theft / Loss

- Laptop ขโมย
- Phone หาย
- USB drive ตก

### 5. Fire / Flood / Disaster

- ไฟไหม้ออฟฟิศ → server ไหม้
- น้ำท่วม → hardware ทั้งห้องเสีย
- แผ่นดินไหว → datacenter ใกล้ฐานเหตุ

### 6. Cloud Provider Outage / Account Lockout

- Google เคย lock account user ที่ "looks suspicious" → user เสีย Gmail + Drive ทั้งหมด
- Service ปิดตัว — เช่น Google Reader, Picasa
- Provider บังคับ migrate รุ่นใหม่ — บางครั้งขาดข้อมูล

### 7. Data Corruption

- Database corruption จาก bug
- Filesystem corruption จาก power loss
- Bit rot — bit สลับเองตามเวลา (ในเครื่องเก่ามาก)

### 8. Software Bug

- Application bug ลบ data
- Sync ผิดทาง — เครื่องที่ลบ sync ไป cloud → ลบใน cloud ด้วย
- Update ผิดทำลาย config

### สรุป

> **ข้อมูลของคุณ "จะหาย" — เป็นเรื่องของ "เมื่อ" ไม่ใช่ "ถ้า"**

ดังนั้น backup ไม่ใช่ optional — แต่เป็นพื้นฐาน

---

## กฎ 3-2-1 Backup — มาตรฐานที่ใช้กันทั่วโลก

หลักการที่ง่ายและทรงพลังที่สุด:

> **3 copies of data, on 2 different media types, with 1 copy off-site**

### 3: Three Copies

ข้อมูลของคุณต้องมี **3 ชุด** เสมอ:

```
Copy 1: Original (ใน laptop / phone / server)
Copy 2: Local backup (external HDD, NAS)
Copy 3: Off-site backup (cloud, สาขาอื่น)
```

ถ้ามีแค่ 1 copy → 1 ครั้งที่หาย = ไม่มีอะไรเหลือ  
ถ้ามี 2 copies → ก็ยังเสี่ยง (เผื่อทั้งคู่อยู่ใกล้กัน — ไฟไหม้ก็พังพร้อม)  
3 copies = ความปลอดภัยที่สมเหตุสมผล

### 2: Two Different Media Types

ใช้ **media ที่ต่างกัน** เพราะ:

- HDD พังในแบบหนึ่ง
- SSD พังในแบบหนึ่ง
- Cloud มีปัญหาคนละแบบ
- Tape (สำหรับ enterprise) เก็บได้นาน

ตัวอย่าง:

- ✅ Laptop SSD + External HDD + Cloud (3 media types)
- ❌ Laptop SSD + Another SSD ใน computer เดียวกัน (1 type, ใกล้กัน)

### 1: One Off-site / Offline Copy

อย่างน้อย **1 copy** ต้องอยู่:

- **Off-site** — ที่อื่นทางภูมิศาสตร์ (กัน fire/flood/theft ที่บ้าน/ออฟฟิศ)
- **Off-line** — ไม่ติดต่อ network (กัน ransomware ที่กระจายผ่าน network)

ตัวอย่าง:
- Cloud backup (off-site แต่ online)
- External HDD ใน safety deposit box ที่ธนาคาร (off-site + offline)
- Tape backup (offline)

### 3-2-1-1-0 — รุ่นปรับปรุงสำหรับยุค Ransomware

หลังยุค ransomware — กฎใหม่:

```
3 copies
2 different media types
1 off-site
1 offline / immutable / air-gapped
0 errors after verification (test restore!)
```

**Immutable backup** = backup ที่ admin ก็ลบไม่ได้ — ป้องกัน ransomware ที่ได้ admin credential

---

## ประเภทของ Backup

### Full Backup

Copy **ทั้งหมด** ของข้อมูลทุกครั้ง

**ข้อดี:**
- Restore ง่าย — ใช้ backup ตัวเดียว
- Self-contained

**ข้อเสีย:**
- ใช้พื้นที่เยอะ (ทุกครั้งเท่าเดิม)
- ใช้เวลานาน
- ต้นทุน storage สูง

**เมื่อใช้:** Weekly / Monthly snapshot, complete archive

### Incremental Backup

Copy **เฉพาะที่เปลี่ยน** ตั้งแต่ backup ครั้งล่าสุด (ไม่ว่าจะ full หรือ incremental)

```
Sunday:    Full backup        (300GB)
Monday:    Incremental       (5GB - ที่เปลี่ยน since Sunday)
Tuesday:   Incremental       (3GB - ที่เปลี่ยน since Monday)
Wednesday: Incremental       (2GB - ที่เปลี่ยน since Tuesday)
```

**ข้อดี:**
- ใช้พื้นที่น้อย
- Backup เร็ว

**ข้อเสีย:**
- Restore ช้า — ต้องเอา full + ทุก incremental มาผสม
- ถ้า incremental ใดๆ ตั้งแต่ full หาย → restore ต่อไม่ได้

### Differential Backup

Copy **ทั้งหมดที่เปลี่ยนตั้งแต่ full ครั้งล่าสุด**

```
Sunday:    Full backup        (300GB)
Monday:    Differential      (5GB - ที่เปลี่ยน since Sunday)
Tuesday:   Differential      (8GB - ที่เปลี่ยน since Sunday) ← ใหญ่กว่า incremental
Wednesday: Differential      (10GB - ที่เปลี่ยน since Sunday)
```

**ข้อดี:**
- Restore เร็ว — แค่ full + differential ล่าสุด
- ปลอดภัยกว่า incremental (ไม่ chain)

**ข้อเสีย:**
- ใช้พื้นที่มากกว่า incremental

### กลยุทธ์ที่นิยม

**Grandfather-Father-Son (GFS):**

```
Daily:     Incremental (Mon-Sat) → keep 7 days
Weekly:    Full (Sunday) → keep 4 weeks
Monthly:   Full (last Sun of month) → keep 12 months
Yearly:    Full (Dec 31) → keep 7 years
```

ทำให้ recovery ได้ทั้งจาก "เมื่อวาน" และ "ปีก่อน"

---

## Backup สำหรับ Developer

### Git ≠ Backup

นี่เป็นเรื่องที่ dev มือใหม่เข้าใจผิด:

> **"ฉันมี code อยู่ใน GitHub แล้ว — ไม่ต้อง backup"**

**ผิด** ครับ — Git/GitHub เป็น **version control** ไม่ใช่ **backup**

#### เหตุผลที่ Git อย่างเดียวไม่พอ

1. **GitHub account ถูก disable** — Microsoft (ปัจจุบันเจ้าของ GitHub) อาจ ban บัญชีคุณ → repo private หาย
2. **Force push ทำลาย history** — `git push --force` แก้ history ได้ → ของเก่าหาย
3. **`.gitignore` ของไฟล์สำคัญ** — env files, secrets, generated files ที่สำคัญ
4. **Submodules / dependencies** — repo external ที่ archive
5. **Repo ถูกลบ** — เผลอ `gh repo delete`
6. **แค่ source code** — ไม่ใช่ environment, infrastructure, data

#### What to backup beyond Git

```
Git repo (source code) → has GitHub
+ Environment configs (.env files) → ⚠️ ต้อง backup แยก (encrypted)
+ Database dumps → ⚠️ ต้อง backup แยก
+ Uploaded files (S3, file storage) → ⚠️ ต้อง backup แยก
+ Infrastructure configs → ⚠️ Terraform state, k8s configs
+ CI/CD secrets → ⚠️ encrypted backup
+ SSH keys → ⚠️ password manager
+ Documentation → ⚠️ ที่ไม่ใช่ in-repo
```

### Database Backup Strategies

#### Logical vs Physical

**Logical backup** — dump ข้อมูลเป็น SQL statements

```bash
# PostgreSQL
pg_dump -h localhost -U user mydb > backup.sql

# MySQL  
mysqldump -u user -p mydb > backup.sql

# MongoDB
mongodump --db mydb --out /backup/
```

**ข้อดี:**
- Cross-version compatible
- Human-readable
- Selective restore (แค่บาง table)
- Smaller (ถ้า compressed)

**ข้อเสีย:**
- Slow ทั้ง backup และ restore
- Database ต้อง online (ไม่ใช่ block-level)

**Physical backup** — copy ไฟล์ของ database ตรงๆ

```bash
# PostgreSQL — pg_basebackup
pg_basebackup -D /backup -X stream -P

# MySQL — Percona XtraBackup
xtrabackup --backup --target-dir=/backup
```

**ข้อดี:**
- เร็วมาก
- ใช้คู่กับ WAL ทำ point-in-time recovery ได้

**ข้อเสีย:**
- ต้องเป็น version เดียวกันถึง restore ได้
- ไฟล์ใหญ่กว่า

#### Point-in-Time Recovery (PITR)

ทำให้ restore กลับไปยัง **เวลาเฉพาะเจาะจง** ได้ — เช่น 2 วินาที ก่อนที่ DBA ลบ table:

```
[Full Backup at 2026-04-29 00:00] 
   + 
[WAL archives from 00:00 to disaster time]
   ↓
Restore to 2026-04-29 14:23:45.123
```

ตั้งใน PostgreSQL:

```bash
# postgresql.conf
archive_mode = on
archive_command = 'cp %p /backup/wal_archive/%f'
wal_level = replica

# ตอน restore
restore_command = 'cp /backup/wal_archive/%f %p'
recovery_target_time = '2026-04-29 14:23:45'
```

#### Backup Schedule ที่แนะนำ

```
Production database:
  - Logical full backup    : daily (off-peak hours)
  - WAL archive            : continuous
  - Point-in-time recovery : enabled
  - Retention              : 30 days
  - Test restore           : weekly (automated)
  - Off-site replication   : real-time to standby DB
```

### Infrastructure as Code = Rebuild Anywhere

แทนที่จะ backup ทั้ง infrastructure เป็น VM image — เก็บ **code ที่สร้าง infrastructure** ใน Git:

- **Terraform** — declarative cloud infra
- **Pulumi** — programmatic cloud infra
- **Ansible** — configuration management
- **Kubernetes manifests** — application deployments

```hcl
# Terraform example
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.medium"
  
  tags = {
    Name = "web-server"
  }
}
```

ถ้า region ทั้ง region ล้ม:

```bash
terraform apply -var="region=us-west-2"
# Provision ทุกอย่างใหม่ในอีก region ภายในนาที
```

**ข้อสำคัญ:**
- เก็บ **Terraform state** ใน remote backend ที่ versioned (S3 + DynamoDB locking)
- **Encrypt** state file (มี secrets)
- Separate environments — dev / staging / prod แยก state

### Application Backup Stack

```
[Source code]      → GitHub (+ mirror to GitLab/Bitbucket)
[Database]         → pg_basebackup + WAL → S3 (+ cross-region replication)
[User uploads/S3]  → S3 versioning + cross-region replication
[Secrets]          → AWS KMS / Vault (+ encrypted backup)
[Infrastructure]   → Terraform in Git
[Logs]             → CloudWatch / Datadog (+ S3 archive)
[Monitoring data]  → Prometheus snapshots → S3
[CI/CD configs]    → GitHub Actions YAML in repo
```

### Test Restore — สำคัญที่สุด

**Backup ที่ไม่ได้ทดสอบ — ไม่ใช่ backup**

GitLab incident แสดงให้เห็นว่า — backup จะ silently fail ตลอด คุณต้อง **ทดสอบ** เป็นประจำ

#### Automated Restore Testing

```python
# pseudo-code สำหรับ automated test
def test_backup_restore():
    # 1. Spin up empty database
    test_db = create_temp_database()
    
    # 2. Restore from latest backup
    restore_backup(test_db, latest_backup)
    
    # 3. Run integrity checks
    assert query("SELECT COUNT(*) FROM users") > 0
    assert query("SELECT MAX(created_at) FROM users") > yesterday()
    
    # 4. Compare row counts with production
    prod_count = production_query("SELECT COUNT(*) FROM users")
    backup_count = query("SELECT COUNT(*) FROM users")
    assert abs(prod_count - backup_count) < tolerance
    
    # 5. Cleanup
    delete_temp_database(test_db)
    
    # 6. Alert if any check fails
```

ตั้งให้รันทุกสัปดาห์ — alert ถ้า fail

---

## Backup สำหรับชีวิตส่วนตัว

ไม่ใช่แค่ dev — ทุกคนต้องมี backup ส่วนตัว

### What to Backup

#### High Priority — สูญเสียไม่ได้

- **รูปภาพ + วิดีโอ** ของครอบครัว — ไม่มีทางถ่ายใหม่
- **เอกสารราชการ scan** — บัตรประชาชน, passport, license
- **เอกสารทางการเงิน** — slip โอนเงิน, สัญญา, ใบรับรอง
- **Master password / 2FA recovery** — ที่กล่าวในบทก่อนๆ

#### Medium Priority

- เอกสารงาน
- Notes ส่วนตัว
- Email ที่สำคัญ
- Contact list

#### Low Priority — มีก็ดี

- Music collection (ส่วนใหญ่ stream ได้)
- Movies (ดู streaming ใหม่ได้)
- Game saves

### Backup Layers สำหรับคนทั่วไป

#### Layer 1: Auto-Sync to Cloud (เริ่มเลย)

เปิด built-in cloud sync ของ device:

**Apple ecosystem:**
- iCloud Photos — sync รูปอัตโนมัติ
- iCloud Drive — sync เอกสาร
- iCloud Backup (iPhone) — เปิดให้ backup auto ตอนชาร์จ + WiFi

**Android:**
- Google Photos — backup รูป
- Google Drive — sync เอกสาร
- Google One backup (มือถือ)

**Windows:**
- OneDrive — sync เอกสาร + Desktop
- Microsoft account สำหรับ password sync

✅ Auto, no maintenance — แต่ Layer 1 อย่างเดียวไม่พอ (ถ้า account ถูก lock = หาย)

#### Layer 2: External HDD (Local Backup)

ซื้อ external HDD 1-2 TB (ราคาประมาณ 1,500-3,000 บาท)

**Mac:** Time Machine — built-in, set-and-forget

```
System Settings → General → Time Machine → 
   Select Backup Disk
```

**Windows:** File History หรือ Windows Backup

```
Settings → Update & Security → Backup → 
   Add a drive
```

**Manual sync** ทุกสัปดาห์/เดือน — รูปจากมือถือ → external HDD

#### Layer 3: Off-site Backup (ป้องกันไฟ/น้ำ/ขโมย)

เลือกอย่างใดอย่างหนึ่ง:

**ตัวเลือก A: Cloud Backup Service**

- **Backblaze** — $9/month, unlimited storage, set-and-forget
- **iDrive** — multi-device, $5/month for 1TB
- **Arq** — pay once, use any cloud (S3, B2, etc.)

**ตัวเลือก B: External HDD ในที่อื่น**

- HDD 2 ตัว — ตัวที่ 2 ฝากบ้านพ่อแม่ / safety deposit box
- Sync เป็นระยะ (ทุก 3-6 เดือน)

**ตัวเลือก C: NAS ที่บ้าน + Cloud Sync**

- **Synology** / **QNAP** NAS — เริ่มต้น 8,000-15,000 บาท
- มี RAID + auto cloud sync
- เหมาะกับครอบครัว

#### My Personal Setup (ตัวอย่าง)

```
รูปจากมือถือ:
  Phone → iCloud Photos (Layer 1) → Backblaze backup (Layer 3)

เอกสารสำคัญ:
  Mac → iCloud Drive → Time Machine (Layer 2) → Backblaze (Layer 3)

ของที่ scan:
  Mac (encrypted folder) → Time Machine → Backblaze

Password manager:
  1Password (cloud sync) → recovery key พิมพ์ + เก็บใน safety deposit box
```

### มือถือ — กลยุทธ์เฉพาะ

#### iPhone

```
Settings → [your name] → iCloud → iCloud Backup → ON
+ iCloud Photos → ON
+ iCloud Drive → ON
```

📌 **ลองทดสอบ restore** — ต่อ iPhone เปล่า → restore from iCloud → ดูว่ารูปครบไหม

#### Android

```
Settings → System → Backup → Google One backup → ON
+ Google Photos auto-backup → ON  
```

📌 **ลอง factory reset** เครื่อง → restore → ดูว่าครบ (ลองในเครื่องสำรอง ไม่ใช่เครื่องหลัก!)

---

## Disaster Recovery Plan

### RTO และ RPO — ตัวเลขที่ต้องรู้

#### RTO — Recovery Time Objective

> **"นานแค่ไหนที่ business จะกลับมาทำงานได้?"**

ถ้า disaster เกิดขึ้น 14:00 → restore เสร็จ 14:30 → RTO = 30 นาที

#### RPO — Recovery Point Objective

> **"เสียข้อมูลได้กี่ชั่วโมง?"**

Backup ล่าสุดเสร็จ 13:00 → disaster เกิด 14:00 → ข้อมูลของ 13:00-14:00 หาย → RPO = 1 ชั่วโมง

### Examples ของ RTO/RPO

| ระบบ | RTO | RPO |
|---|---|---|
| Email server (internal) | 4 ชั่วโมง | 1 ชั่วโมง |
| Customer-facing website | 1 ชั่วโมง | 5 นาที |
| Banking core system | 30 นาที | 0 (zero-data-loss) |
| Marketing CMS | 24 ชั่วโมง | 24 ชั่วโมง |
| Personal photos | 1 สัปดาห์ | 1 วัน |

**ค่า RTO/RPO ส่งผลต่อ cost** — ยิ่งต่ำ ยิ่งแพง

- **RPO = 0** ต้องการ synchronous replication (สองเท่าของ infra)
- **RPO = 1 hour** ต้องการ continuous backup
- **RPO = 24 hours** ต้องการ daily backup เพียงพอ

### Disaster Recovery Tiers

#### Tier 0: No Off-site Plan
- Backup ในที่เดียวกับ production
- Disaster ใกล้เคียง = หมดสิ้น

#### Tier 1: Pickup Truck Access Method
- Tape backup ที่ขนไปอีกที่
- RTO: หลายวัน-สัปดาห์

#### Tier 2: Hot Site
- มี standby infrastructure ที่อีก site
- Manual restore from backup
- RTO: ชั่วโมง-วัน

#### Tier 3: Active-Passive
- Production ที่ site A
- Standby DB sync to site B
- Failover manual
- RTO: นาที-ชั่วโมง

#### Tier 4: Active-Active
- ทั้ง 2 sites รับ traffic พร้อมกัน
- Auto failover ถ้า site ใดล้ม
- RTO: วินาที-นาที

#### Tier 5: Real-time Multi-region
- Cross-region active-active
- Auto-routing ของ traffic
- RTO: ใกล้ 0
- ต้นทุนสูงมาก — สำหรับ critical infrastructure

### DR Drill — ซ้อมเป็นประจำ

ใช่ครับ — แม้คุณมี DR plan ที่สวยงาม **ถ้าไม่ซ้อม = ใช้ไม่ได้จริง**

#### Tabletop Exercise

นั่งคุยกันในห้อง — สมมติ disaster แล้วอธิบายว่าจะทำยังไง

#### Functional Drill

เริ่ม activate DR plan จริง — ทดสอบ component บางส่วน

#### Full-scale Drill

Failover production → DR site → run จริงๆ → failback

แนะนำ:

- **Tabletop:** ทุก 6 เดือน
- **Functional:** ทุกปี
- **Full-scale:** อย่างน้อยทุก 2 ปี

---

## Hands-on: Setup Backup ทันที

### สำหรับคนทั่วไป (15 นาที)

#### 1. เปิด Cloud Backup

**iPhone:**
```
Settings → [name] → iCloud → 
   iCloud Backup → ON
   iCloud Photos → ON
   iCloud Drive → ON
```

**Android:**
```
Settings → System → Backup → 
   Backup by Google One → ON
   + Google Photos auto-backup
```

**Mac:**
```
System Settings → General → Time Machine → 
   Add Backup Disk
```

**Windows 11:**
```
Settings → Accounts → Windows Backup → 
   Sync to OneDrive
```

#### 2. ซื้อ External HDD

ขนาด:
- 1 TB สำหรับเอกสาร (~1,500 บาท)
- 2 TB สำหรับเอกสาร + รูป + วิดีโอ (~2,500 บาท)
- 4 TB ขึ้นไปถ้ามีวิดีโอเยอะ (~4,000+ บาท)

แนะนำยี่ห้อ: WD My Passport, Seagate Backup Plus, Toshiba Canvio

#### 3. Setup Time Machine / File History

**Mac (Time Machine):**
- เสียบ HDD → Mac ถาม "use as Time Machine?" → Yes
- Auto backup ทุกชั่วโมง (ตอนเสียบ)

**Windows (File History):**
```
Settings → Update & Security → Backup → 
   Add a drive → เลือก HDD
```

#### 4. Setup Off-site Backup

ตัวเลือก:

**Backblaze ($9/month):**
- [backblaze.com](https://www.backblaze.com)
- ติดตั้ง app → set-and-forget
- Unlimited storage บน computer

**iDrive (~$5/month for 1TB):**
- Multi-device — มือถือ + computer
- Encrypted

#### 5. Test Restore

**ทุก 3 เดือน:**
- ลอง restore 1 ไฟล์จาก Time Machine
- ลอง download 1 ไฟล์จาก iCloud / Google Drive
- ถ้า restore ได้ — confirm backup ใช้ได้

### สำหรับ Developer / Small Team (1 hour)

#### 1. Database Backup Automation

```bash
# /etc/cron.d/db-backup
0 2 * * * postgres pg_dump -Fc mydb | gzip > /backup/mydb-$(date +\%F).sql.gz

# Sync to S3
0 3 * * * aws s3 sync /backup s3://my-backup-bucket/db/ --delete
```

S3 bucket settings:
- ✅ Versioning enabled
- ✅ Object lock (immutable)
- ✅ Cross-region replication
- ✅ Lifecycle: move to Glacier after 30 days

#### 2. Source Code Mirror

```bash
# Daily mirror to GitLab/Bitbucket
0 4 * * * cd /repo && git push --mirror gitlab.com/myorg/myrepo
```

#### 3. Infrastructure as Code

ทุก infra → Terraform / Pulumi → Git

#### 4. Test Restore Script

```bash
# Weekly automated test
0 6 * * 0 /scripts/test-backup-restore.sh && \
  curl -X POST https://hooks.slack.com/...   # alert if fails
```

---

## Action Items

### วันนี้ (ทุกคน)

- [ ] เปิด **iCloud Backup / Google One Backup** บนมือถือ
- [ ] เปิด **Time Machine / File History** บน laptop
- [ ] ซื้อ external HDD ถ้ายังไม่มี

### สัปดาห์นี้ (ทุกคน)

- [ ] **Test restore** อย่างน้อย 1 ไฟล์จาก backup
- [ ] **เลือก cloud backup service** + subscribe
- [ ] **Document** ว่าข้อมูลอะไรอยู่ที่ไหน

### สำหรับ Developer

- [ ] ตรวจสอบ **database backup** ทำงานจริงไหม
- [ ] ตรวจ **WAL/binlog archive** เปิดอยู่ไหม
- [ ] **Automate test restore** weekly
- [ ] **Source code mirror** ที่อื่นนอก GitHub
- [ ] **Terraform state** อยู่ใน remote backend ที่ versioned + encrypted
- [ ] **Document RTO/RPO** ของแต่ละ service

### สำหรับ Tech Lead / IT Manager

- [ ] **Backup policy document** ที่ทุกคนรู้
- [ ] **DR plan documented**
- [ ] **DR drill schedule** — tabletop, functional, full-scale
- [ ] **Budget for backup infrastructure** — บ่อยถูกตัดเพราะ "ไม่เห็นว่าจำเป็น"
- [ ] **Vendor contracts** สำหรับ DR site / cloud failover
- [ ] **Cyber insurance** ที่ครอบคลุม disaster

---

## บทเรียนชีวิตจากบทความนี้

> **คนฉลาดเตรียมร่มก่อนฝนตก — backup คือร่มของโลกดิจิทัล**

ในชีวิตเราก็มี "backup" ในรูปแบบต่างๆ:

- **สำเนาเอกสาร** — บัตรประชาชน, passport, license — เก็บ scan ไว้
- **พินัยกรรม** — backup ของการตัดสินใจถ้าเราไม่อยู่
- **ประกันชีวิต/สุขภาพ** — financial backup เมื่อเกิดเรื่อง
- **เงินสำรอง 6 เดือน** — emergency fund สำหรับวิกฤต
- **ทักษะหลายๆ ด้าน** — career backup ถ้าตำแหน่งหายไป
- **เครือข่ายเพื่อน mentor** — social backup ตอนเหงา / เครียด
- **ความสัมพันธ์ที่ดีกับครอบครัว** — emotional backup

> **คนที่ไม่เคยเตรียม backup ในชีวิต เมื่อ "ฝนตก" ก็ "เปียกหมดตัว"**

ในขณะที่บางคนคิดว่า:

- "ไม่หรอก ไม่มีทางเกิดขึ้น"
- "ไว้ค่อยทำตอนจำเป็น"
- "ถ้าเกิดขึ้น ค่อยคิด"

จนวันที่จำเป็นมาถึง — แล้วไม่มีอะไรเตรียม

> **บทเรียนของ GitLab สอนว่า — มี backup 5 อย่าง อาจจะเหลือ 1 ที่ใช้ได้  
> ถ้าไม่มีเลย = หมดทุกอย่าง**

ในชีวิตก็เช่นกัน — มี backup เยอะๆ ไม่ได้แปลว่าจะใช้ทุกอัน แต่ใช้แค่ตัวเดียวก็เพียงพอที่จะรอดในวันวิกฤต

3 หลักการที่ใช้ได้ทั้งใน data และในชีวิต:

1. **Diversify** — กระจายความเสี่ยง อย่าใส่ทุกอย่างในตะกร้าใบเดียว
2. **Test regularly** — ใช้งานเป็นระยะ ไม่ใช่ "มีไว้เผื่อ" แล้วลืม
3. **Plan for failure** — ไม่ใช่ "ถ้าล้ม" แต่ "เมื่อล้ม"

> **ผู้ใหญ่ที่จริงๆ — คือคนที่เตรียมตัวสำหรับวันที่ไม่อยากให้มาถึง**

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไปเราเข้าสู่ Part 5 — **Developer-Specific** เรื่องที่ dev ต้องรู้เป็นพิเศษ เริ่มจาก **Secrets Management** — อย่า hardcode ความลับลงใน code ครับ

---

## อภิธานศัพท์ (Glossary)

- **Backup:** สำเนาของข้อมูลเก็บไว้กรณีต้นฉบับหาย
- **3-2-1 Rule:** 3 copies, 2 media types, 1 off-site
- **Immutable Backup:** backup ที่แก้ไขไม่ได้แม้แต่ admin
- **Air-gapped Backup:** disconnected จาก network โดยสิ้นเชิง
- **Full Backup:** copy ทั้งหมดของข้อมูล
- **Incremental Backup:** copy เฉพาะที่เปลี่ยน since last backup
- **Differential Backup:** copy ที่เปลี่ยน since last full backup
- **GFS (Grandfather-Father-Son):** rotation strategy ของ backup
- **WAL (Write-Ahead Log):** log ของ database ที่ใช้ replay สำหรับ PITR
- **PITR (Point-in-Time Recovery):** restore กลับไปยังเวลาเฉพาะเจาะจง
- **Logical Backup:** dump เป็น SQL/JSON/etc.
- **Physical Backup:** copy block-level ของ database files
- **RTO (Recovery Time Objective):** เวลาที่ยอมรับได้สำหรับ restore
- **RPO (Recovery Point Objective):** ข้อมูลที่ยอมรับได้ว่าหาย
- **Hot Site:** DR site ที่ active พร้อมใช้
- **Cold Site:** DR site ที่ต้อง provision
- **Active-Active:** ทั้งสอง site รับ traffic
- **Active-Passive:** primary รับ traffic, secondary standby
- **Failover:** สลับจาก primary → secondary
- **Failback:** กลับมาใช้ primary หลัง recover
- **Tabletop Exercise:** ซ้อม DR แบบสมมติ ไม่ใช้ระบบจริง
- **Bit Rot:** data corruption จาก storage degradation
- **IaC (Infrastructure as Code):** infra ที่กำหนดด้วย code (Terraform, Pulumi)

---

## สรุป

1. **Backup ไม่ใช่แค่เรื่อง ransomware** — hardware fail, human error, theft, fire ก็ทำลายข้อมูลได้
2. **3-2-1 Rule:** 3 copies, 2 media types, 1 off-site (+ 1 immutable + 0 errors after testing)
3. **Full vs Incremental vs Differential** — เลือกใช้ตาม use case + storage budget
4. **Git ≠ backup** — backup ต้องมี database, configs, infrastructure, secrets ด้วย
5. **PITR + WAL archive** สำหรับ database ที่ critical
6. **IaC ทำให้ rebuild ได้** — Terraform / Pulumi
7. **Backup ที่ไม่ทดสอบ = ไม่ใช่ backup** — automated restore testing
8. **RTO/RPO** กำหนด tier ของ DR
9. **Personal backup** = cloud + external HDD + cloud backup service
10. **DR drill** เป็นประจำ — ไม่ซ้อม = ใช้ไม่ได้จริง

ใน Part ถัดไป (Developer-Specific) เราจะเริ่มที่ **Secrets Management** — ความลับใน code, hardcoded API keys, vault solutions และเรื่องที่ dev พลาดบ่อยที่สุด

— Claude Opus 4.6
