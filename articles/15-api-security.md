# บทความที่ 15: API Security — ปิดประตูหลังก่อนที่จะสาย

**CyberSecurity Awareness Series — Part 5: Developer-Specific**  
เขียนโดย Claude Opus 4.6 | บรรณาธิการ: Tanin T.

---

## เรื่องเล่าของ API ที่ "เปิดเผยข้อมูล 37 ล้านคน"

เดือนกรกฎาคม ปี 2021 — researchers พบว่า API ของ **T-Mobile USA** มีช่องโหว่ใหญ่ที่:

- Mobile app ของ T-Mobile call API endpoint หนึ่งเพื่อดูข้อมูล user
- API ใช้ **customer ID** ใน URL: `/api/customer/{id}`
- API **ไม่ได้เช็ค** ว่า user ที่ login เป็นเจ้าของ ID นั้นจริงไหม
- ผู้โจมตีแค่ loop ID ตั้งแต่ 1 ขึ้นไป → ดึงข้อมูลของ **37+ ล้าน customer**:
  - ชื่อ, address, date of birth
  - Phone number
  - SSN (social security number)
  - Driver's license

ผู้โจมตีขายข้อมูลเหล่านี้บน dark web ในราคา **$200,000 USD** สำหรับ 30 ล้าน records

[อ่านรายละเอียด T-Mobile breach](https://www.bleepingcomputer.com/news/security/t-mobile-says-breach-exposed-personal-info-of-378-million-customers/)

T-Mobile โดน **ค่าปรับ $350 million** + ต้องลงทุนอีก **$150 million** ใน security

ที่น่าตกใจที่สุด — ช่องโหว่นี้คือ **BOLA (Broken Object Level Authorization)** — bug ที่ basic ที่สุดในวงการ API security — และมันอยู่ใน production ของ telco ใหญ่ที่สุดอันดับ 2 ของอเมริกา

> **API คือประตูหน้าของระบบสมัยใหม่ — ถ้าคุณไม่ปิด ทุกอย่างข้างในเปิดให้ดู**

ในยุคปี 2026 — ทุก mobile app, ทุก SPA, ทุก microservice คุยกันผ่าน API การ secure API ไม่ใช่ optional แต่เป็นพื้นฐานของระบบทุกชนิด

---

## Authentication vs Authorization — เข้าใจให้ชัด

ก่อนเริ่ม — ต้องแยกสองคำนี้ให้ออก เพราะ dev จำนวนมาก **สับสน** และทำให้ระบบรั่ว:

### Authentication (AuthN) = "คุณเป็นใคร?"

ตรวจสอบว่า request นี้ **มาจากใคร** — login, JWT verification, API key validation

```python
# Authentication: who are you?
@app.before_request
def authenticate():
    token = request.headers.get('Authorization')
    if not token:
        abort(401)  # Unauthorized
    
    user = verify_jwt(token)
    if not user:
        abort(401)
    
    g.current_user = user  # ตอนนี้ระบบรู้ว่าใคร
```

### Authorization (AuthZ) = "คุณทำได้ไหม?"

ตรวจสอบว่า user ที่ authenticated แล้ว **มีสิทธิ์** ทำ action นี้ไหม

```python
# Authorization: can you do this?
@app.route('/api/admin/users')
def admin_users():
    if not g.current_user.is_admin:
        abort(403)  # Forbidden
    return get_all_users()
```

### ความสับสนที่พบบ่อย

```python
# ❌ ผิดพลาดที่ใหญ่ที่สุดในวงการ API
@app.route('/api/users/<user_id>')
def get_user(user_id):
    if not g.current_user:  # ✅ Authentication checked
        abort(401)
    
    return User.query.get(user_id).to_dict()
    # ❌ ไม่มี Authorization check!
    # → User คนใดก็ดูข้อมูลของ User คนอื่นได้
    # นี่คือ bug ของ T-Mobile
```

```python
# ✅ ถูกต้อง
@app.route('/api/users/<user_id>')
def get_user(user_id):
    if not g.current_user:
        abort(401)
    
    if g.current_user.id != user_id and not g.current_user.is_admin:
        abort(403)  # Authorization check!
    
    return User.query.get(user_id).to_dict()
```

> **กฎ: ทุก endpoint ต้องเช็ค ทั้ง AuthN และ AuthZ**

### HTTP Status Codes ที่ถูกต้อง

| Status | Meaning |
|---|---|
| **401 Unauthorized** | ไม่มี credentials หรือ credentials ผิด (Authentication ล้มเหลว) |
| **403 Forbidden** | มี credentials แต่ไม่มีสิทธิ์ทำ (Authorization ล้มเหลว) |
| **404 Not Found** | Resource ไม่มี (หรือบางครั้งจงใจ return 404 แทน 403 เพื่อไม่เปิดเผย existence) |

---

## OAuth 2.0 / OpenID Connect Overview

**OAuth 2.0** คือ standard สำหรับ **delegated authorization** — ให้ third-party app เข้าถึง resource ของ user **โดยไม่ต้อง share password**

### ปัญหาที่ OAuth แก้

ก่อน OAuth — ถ้าแอปอยากเข้าถึง Gmail ของคุณ ต้องขอ password Gmail → คุณต้องเชื่อใจ app นั้นเต็มๆ

ปัจจุบัน:

```
User → "Sign in with Google" → 
  Google login page → 
  "Allow App X to read your email?" → 
  Approve → 
  App X ได้ access token (ไม่ใช่ password)
```

App X **ไม่เคยเห็น password** ของคุณ + คุณ revoke access ได้ทุกเมื่อ

### OAuth 2.0 Flow Types (Grant Types)

#### 1. Authorization Code Flow (สำหรับ web apps)

```
[User] → [Web App] → [Auth Server (Google)] 
                         ↓
                    Login + Approve
                         ↓
[Web App] ← (auth code) ←
   ↓
[Web App] → /token endpoint → (access_token + refresh_token)
                                ↑
                         ใช้ client_secret
```

**Use case:** server-side app ที่เก็บ secret ปลอดภัยได้

#### 2. Authorization Code with PKCE (สำหรับ mobile/SPA)

PKCE (Proof Key for Code Exchange) ป้องกัน auth code interception attack สำหรับ public client (ที่เก็บ secret ไม่ได้)

**Use case:** mobile apps, SPAs (React/Vue/Angular)

#### 3. Client Credentials Flow (machine-to-machine)

```
[Service A] → /token endpoint with client_id + client_secret → access_token
```

**Use case:** backend service เรียก API อื่น (ไม่มี user)

#### 4. Refresh Token Flow

Access token มักหมดอายุเร็ว (15 นาที - 1 ชั่วโมง) — ใช้ refresh token แลก access token ใหม่:

```
Access token expired → use refresh_token → new access_token
```

**Use case:** keep user logged in โดยไม่ต้อง re-login

#### 5. Implicit Flow (deprecated)

❌ **อย่าใช้** — มี security flaws ที่ Authorization Code with PKCE แทนได้

#### 6. Resource Owner Password Credentials (deprecated)

❌ **อย่าใช้** — ทำให้ password ส่งผ่าน app

### OpenID Connect (OIDC)

**OIDC** = OAuth 2.0 + Identity layer

OAuth บอกว่า "app นี้มีสิทธิ์ทำอะไร" — แต่ไม่ได้บอกว่า user เป็นใคร

OIDC เพิ่ม **id_token** ที่บอกข้อมูล user (name, email, sub):

```json
{
  "iss": "https://accounts.google.com",
  "sub": "1234567890",
  "aud": "myapp-client-id",
  "exp": 1672531200,
  "name": "Tanin T.",
  "email": "tanin@example.com"
}
```

→ ใช้ทำ "Sign in with Google" / Apple / Microsoft / etc.

### Common Mistakes

#### 1. Validate id_token ผิด

```javascript
// ❌ Decode without verify
const payload = JSON.parse(atob(idToken.split('.')[1]));
const userId = payload.sub;
// → ใครก็แก้ token ได้

// ✅ Verify signature + claims
import { verify } from 'jsonwebtoken';
const payload = verify(idToken, publicKey, {
    issuer: 'https://accounts.google.com',
    audience: 'myapp-client-id',
});
```

#### 2. Trust `state` Parameter ไม่พอ

```python
# ✅ State parameter ป้องกัน CSRF ใน OAuth flow
state = secrets.token_urlsafe(32)
session['oauth_state'] = state

auth_url = f"https://provider.com/authorize?...&state={state}"

# ตอน callback
@app.route('/oauth/callback')
def callback():
    if request.args.get('state') != session['oauth_state']:
        abort(403)
```

#### 3. Implicit Flow ใน SPA

ถ้า SPA ของคุณยังใช้ implicit flow — migrate เป็น Authorization Code with PKCE

#### 4. Long-lived Access Token

```python
# ❌
access_token_expiry = 30 * 24 * 3600  # 30 days

# ✅
access_token_expiry = 15 * 60   # 15 minutes
refresh_token_expiry = 30 * 24 * 3600  # 30 days
```

---

## API Key Management & Scoping

### API Key vs OAuth Token

- **API Key** = static credential สำหรับ server-to-server / service identification
- **OAuth Token** = dynamic credential ที่หมดอายุ + scoped ต่อ user

### Best Practices สำหรับ API Keys

#### 1. Scoped Keys

```python
# ❌ Single super-key
master_api_key = "<your-master-key>"  # access ทุกอย่าง

# ✅ Multiple scoped keys
analytics_read_key = "<analytics-read-key>"   # read-only analytics
billing_admin_key = "<billing-admin-key>"      # billing access
webhook_send_key = "<webhook-send-key>"        # outgoing webhooks
```

#### 2. Per-customer / Per-environment Keys

```python
# Customer A
api_key_customer_a_prod = "<prefix-a-prod-key>"
api_key_customer_a_test = "<prefix-a-test-key>"

# Customer B
api_key_customer_b_prod = "<prefix-b-prod-key>"
```

→ Key รั่ว = revoke เฉพาะ customer / environment นั้น

#### 3. Show Only at Generation Time

```
[User clicks "Generate API Key"]
   ↓
[Show key once: "abc123..."]
   ↓
[User copies key]
   ↓
[ถ้าเข้ามาดูอีก = เห็นแค่ "abc1...****"]
```

→ Server เก็บแค่ hash ของ key — ถ้ารั่วจาก DB = ผู้โจมตียังต้องแฮก hash

```python
import secrets, hashlib

def generate_api_key():
    raw_key = "sk_" + secrets.token_urlsafe(32)
    key_hash = hashlib.sha256(raw_key.encode()).hexdigest()
    
    # Save key_hash ลง DB, return raw_key ให้ user ครั้งเดียว
    db.api_keys.insert({"hash": key_hash, "user_id": user.id})
    return raw_key

def verify_api_key(raw_key):
    key_hash = hashlib.sha256(raw_key.encode()).hexdigest()
    return db.api_keys.find_one({"hash": key_hash})
```

#### 4. Show Prefix สำหรับ Identification

```
sk_live_abc12345...xyz
^^^^^^^                  ← ดู prefix รู้ว่าใช้ที่ไหน
       ^^^^             ← แสดงตอน list ใน UI
            ^^^         ← masked
```

#### 5. Expiration

API keys ไม่ควร valid ตลอดไป — ตั้ง expiration date:

```python
api_key = create_key(
    user_id=user.id,
    expires_at=datetime.utcnow() + timedelta(days=365),
)
```

#### 6. Revocation API

User ต้อง revoke key ได้ทันที:

```python
@app.route('/api/keys/<key_id>/revoke', methods=['POST'])
def revoke_key(key_id):
    key = ApiKey.query.get(key_id)
    if key.user_id != current_user.id:
        abort(403)
    
    key.revoked_at = datetime.utcnow()
    db.session.commit()
    
    # Invalidate cache (ถ้ามี)
    cache.delete(f"api_key:{key.hash}")
```

#### 7. Audit Log

ทุก API key usage → log:

```python
log.info("api_key_used", extra={
    "key_id": key.id,
    "endpoint": request.path,
    "ip": request.remote_addr,
    "user_agent": request.user_agent.string,
    "timestamp": datetime.utcnow().isoformat(),
})
```

→ Detect abnormal patterns + investigate ตอน incident

---

## Rate Limiting & Throttling

API ต้องมี rate limit เสมอ — ไม่งั้น:

- **DDoS** — ผู้โจมตี flood request → server crash
- **Brute force** — ลอง password / token เป็นล้านครั้ง
- **Resource exhaustion** — query ที่หนัก ทำให้ DB ตาย
- **Cost abuse** — ถ้า API คิดเงินตาม request → bill โต

### Rate Limit Strategies

#### 1. Per-IP

```python
from flask_limiter import Limiter

limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["100 per hour", "5 per minute"]
)

@app.route('/api/data')
@limiter.limit("10 per minute")
def get_data():
    ...
```

**ปัญหา:** Mobile users ใช้ NAT — IP เดียวกันหลายคน + IPv6

#### 2. Per-User / Per-API-Key

```python
@limiter.limit("100 per hour", key_func=lambda: g.current_user.id)
def get_data():
    ...
```

**ดีกว่า** — แต่ require authentication

#### 3. Per-Endpoint

```python
# Login endpoint — strict (กัน brute force)
@app.route('/login', methods=['POST'])
@limiter.limit("5 per minute")
def login():
    ...

# Search endpoint — relaxed
@app.route('/search')
@limiter.limit("60 per minute")
def search():
    ...
```

#### 4. Tiered Rate Limit

```python
# Free tier: 100/hour
# Paid tier: 1000/hour
# Enterprise: 10000/hour

def get_rate_limit():
    user = g.current_user
    if user.tier == 'free':
        return "100 per hour"
    elif user.tier == 'paid':
        return "1000 per hour"
    return "10000 per hour"

@app.route('/api/data')
@limiter.limit(get_rate_limit)
def get_data():
    ...
```

### Rate Limit Algorithms

#### Token Bucket

ทุก user มี "bucket" ที่ refill ตามเวลา — ใช้ token ทำ request

#### Leaky Bucket

Request เข้า queue — process ทีละชิ้นตาม rate

#### Fixed Window

นับ request ใน window 1 นาที — reset ทุกนาที

#### Sliding Window

เหมือน fixed แต่ window เลื่อนตามเวลา (smoother)

### Rate Limit Headers

API ที่ดี return headers ให้ client รู้:

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 47
X-RateLimit-Reset: 1672531200
```

ถ้า limit hit:

```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1672531200
Retry-After: 60
```

### Implementation

#### Redis-based (most popular)

```python
import redis
r = redis.Redis()

def is_rate_limited(user_id, endpoint, max_requests=100, window=3600):
    key = f"rate:{user_id}:{endpoint}"
    pipe = r.pipeline()
    pipe.incr(key)
    pipe.expire(key, window)
    count = pipe.execute()[0]
    return count > max_requests
```

#### Cloud-managed

- **AWS API Gateway** — built-in rate limiting + throttling
- **Cloudflare Rate Limiting** — edge-level
- **Azure API Management** — ตัวเลือก rule-based
- **Kong / Tyk** — API gateways

---

## CORS — เข้าใจจริงๆ ว่าป้องกันอะไร

**CORS** (Cross-Origin Resource Sharing) เป็นเรื่องที่ dev สับสนมากที่สุดในวงการ web

### CORS ป้องกันอะไร?

> **CORS = browser feature ที่บล็อก cross-origin AJAX requests by default**

ตัวอย่าง: คุณ login `bank.com` แล้วเปิด `evil.com`:

```javascript
// ใน evil.com
fetch('https://bank.com/api/transfer', {
    method: 'POST',
    body: JSON.stringify({ to: 'attacker', amount: 100000 }),
    credentials: 'include'
});
```

ถ้าไม่มี CORS → browser ส่ง cookie ของ `bank.com` ตามไป → bank.com ทำ transfer

ด้วย CORS — browser **block response** ก่อนที่ JavaScript จะอ่านได้ (ในบางกรณี + preflight ก่อน)

> **CORS ปกป้อง user ของ origin หนึ่ง จาก site อื่นที่ทำ request แทน user**

### CORS ไม่ได้ป้องกันอะไร

#### 1. CORS ไม่ป้องกัน server attack

```bash
# จาก curl / Python — ไม่มี CORS
curl -X POST https://bank.com/api/transfer ...
```

CORS เป็น **browser-only** — backend service เรียก API ปกติได้เสมอ

#### 2. CORS ไม่ป้องกัน CSRF บางแบบ

`<form>` submit ข้าม origin **ไม่ block** โดย CORS

→ ต้องใช้ CSRF token + SameSite cookie ร่วมด้วย

#### 3. CORS ไม่ใช่ authorization

```python
# ❌ คนคิดว่า CORS เป็น "white list"
@app.after_request
def cors(response):
    response.headers['Access-Control-Allow-Origin'] = 'https://myapp.com'
    return response

@app.route('/api/admin/delete-all')
def delete_all():
    db.delete_all()  # ไม่มี auth check
```

CORS แค่บอก browser ว่า "myapp.com อนุญาต" — backend ยังต้องเช็ค auth เอง

### CORS Headers

#### Server response

```http
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 3600
```

#### Preflight Request

ก่อน request ที่ "complex" (PUT, DELETE, custom header) — browser ส่ง OPTIONS request ก่อน:

```http
OPTIONS /api/data HTTP/1.1
Origin: https://myapp.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization
```

Server ตอบ:

```http
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: PUT
Access-Control-Allow-Headers: Authorization
```

### Common Mistakes

#### 1. `Access-Control-Allow-Origin: *` + Credentials

```http
# ❌ Browser block — ไม่ work
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

ถ้า cred = true → ต้อง specify origin เฉพาะ

#### 2. Reflect Origin โดยไม่ check

```python
# ❌ Reflect ทุก origin
@app.after_request
def cors(response):
    response.headers['Access-Control-Allow-Origin'] = request.headers.get('Origin')
    response.headers['Access-Control-Allow-Credentials'] = 'true'
    return response
```

→ **เปิดเผย** เท่ากับไม่มี CORS เลย — เพราะ origin ใดๆ ก็ได้รับอนุญาต

#### 3. Allow-list ที่ regex ผิด

```python
# ❌
allowed = request.headers.get('Origin').endswith('myapp.com')
# myapp.com.evil.com ก็ pass!

# ✅
import re
allowed = re.match(r'^https://(.*\.)?myapp\.com$', request.headers.get('Origin'))
```

---

## Common API Vulnerabilities — OWASP API Top 10

OWASP มี API-specific Top 10:

### 1. BOLA (Broken Object Level Authorization) — IDOR

ที่กล่าวข้างต้น — ลืมเช็คว่า user มีสิทธิ์เข้าถึง object ที่ขอ

### 2. Broken Authentication

- Weak credential validation
- JWT misconfigurations
- Long-lived tokens
- Predictable token

### 3. Broken Object Property Level Authorization

```python
# ❌ Mass assignment
@app.route('/api/users/<user_id>', methods=['PATCH'])
def update_user(user_id):
    user = User.query.get(user_id)
    for key, value in request.json.items():
        setattr(user, key, value)  # ❌ user ส่ง is_admin=true ก็ได้
    db.session.commit()

# ✅ Allow-list of fields
ALLOWED_FIELDS = ['name', 'email', 'avatar_url']

@app.route('/api/users/<user_id>', methods=['PATCH'])
def update_user(user_id):
    user = User.query.get(user_id)
    for key, value in request.json.items():
        if key in ALLOWED_FIELDS:
            setattr(user, key, value)
    db.session.commit()
```

### 4. Unrestricted Resource Consumption

- Missing rate limit
- Large payload accepted
- Expensive query without limit
- File upload without size limit

### 5. Broken Function Level Authorization

```python
# ❌ User สามารถเข้าถึง admin endpoint
@app.route('/api/admin/users')
@require_login  # มี auth แต่ไม่มี role check
def admin_users():
    return User.query.all()
```

### 6. Unrestricted Access to Sensitive Business Flows

- ไม่ rate limit booking system → ใครก็ book ที่นั่งทั้งหมดให้ฟรี
- ไม่ verify ก่อนทำ refund → abuse

### 7. Server-Side Request Forgery (SSRF)

ที่กล่าวในบทที่ 14

### 8. Security Misconfiguration

- Default credentials
- Debug endpoints in production
- Verbose error messages
- Unsecured admin interface

### 9. Improper Inventory Management

- Old API versions ที่ deprecated แต่ยัง online
- Internal API endpoints ที่ผ่าน public

### 10. Unsafe Consumption of APIs

- ไม่ verify response จาก 3rd party API
- ไม่มี timeout
- ไม่มี circuit breaker

### Excessive Data Exposure

```python
# ❌ Return ทุก field
@app.route('/api/users/<user_id>')
def get_user(user_id):
    user = User.query.get(user_id)
    return jsonify(user.__dict__)  # ❌ password_hash, ssn, etc.

# ✅ Allow-list of fields
@app.route('/api/users/<user_id>')
def get_user(user_id):
    user = User.query.get(user_id)
    return jsonify({
        "id": user.id,
        "name": user.name,
        "email": user.email,
        "avatar_url": user.avatar_url,
    })
```

---

## API Versioning + Deprecation

### URL-based

```
/v1/users
/v2/users
```

ง่ายที่สุด, popular ที่สุด

### Header-based

```http
GET /users
API-Version: 2
```

### Content negotiation

```http
GET /users
Accept: application/vnd.myapp.v2+json
```

### Deprecation Strategy

```python
@app.route('/v1/users')
def v1_users():
    response = get_users()
    response.headers['Sunset'] = 'Sat, 31 Dec 2026 23:59:59 GMT'
    response.headers['Deprecation'] = 'true'
    response.headers['Link'] = '</v2/users>; rel="successor-version"'
    return response
```

ตั้ง expiration ให้ user รู้ว่าต้อง migrate

---

## API Documentation = Security

API docs ที่ดี = ลด security incidents

### OpenAPI / Swagger

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
paths:
  /users/{id}:
    get:
      security:
        - BearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: User found
        '401':
          description: Unauthorized
        '403':
          description: Forbidden
        '404':
          description: Not found
components:
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
```

→ Auto-generate client SDK + interactive docs (Swagger UI)

### Documentation Should Include

- Authentication required ไหม
- Rate limit
- Required vs optional fields
- Possible error responses
- Examples
- Versioning strategy
- Deprecation timeline

---

## Hands-on: Audit API ของคุณ

### Checklist

#### Authentication

- [ ] ทุก endpoint require authentication (ยกเว้น public)
- [ ] ใช้ industry standards (JWT, OAuth)
- [ ] Tokens หมดอายุได้
- [ ] Logout invalidate token (server-side)

#### Authorization

- [ ] ทุก endpoint check authorization
- [ ] Resource access verified ownership
- [ ] Role-based access for admin
- [ ] Object-level authorization (BOLA prevention)

#### Input Validation

- [ ] Request body validated
- [ ] Path parameters validated  
- [ ] Query strings validated
- [ ] Headers validated (Content-Type, Authorization format)

#### Rate Limiting

- [ ] Per-IP / Per-user limits
- [ ] Stricter limits for auth endpoints
- [ ] 429 response with Retry-After

#### Error Handling

- [ ] Generic error messages (no DB schema, no stack trace)
- [ ] Consistent error format
- [ ] Log details server-side, return generic to client

#### Logging

- [ ] All API calls logged
- [ ] No sensitive data in logs
- [ ] Logs centralized + searchable
- [ ] Alert on suspicious patterns

#### CORS

- [ ] Specific origins (not `*`)
- [ ] Credentials handling correct
- [ ] Preflight responses correct

#### TLS

- [ ] HTTPS only (no HTTP)
- [ ] HSTS header
- [ ] TLS 1.2+ minimum

#### Versioning

- [ ] Clear versioning strategy
- [ ] Deprecation policy documented
- [ ] Old versions sunset properly

#### Documentation

- [ ] OpenAPI/Swagger spec
- [ ] Authentication flow documented
- [ ] Error responses documented
- [ ] Rate limits documented

---

## Action Items

### วันนี้

- [ ] **List ทุก API endpoint** ในระบบ → check ว่ามี auth + authz ทุกตัว
- [ ] **Test BOLA** — เปลี่ยน ID ใน URL ดูว่าเข้าถึงข้อมูลคนอื่นได้ไหม
- [ ] **Audit response payload** — มี sensitive field ที่ไม่ควร return ไหม

### สัปดาห์นี้

- [ ] **เพิ่ม rate limiting** ใน auth endpoints
- [ ] **Validate ทุก input** — body, params, headers
- [ ] **Setup OpenAPI documentation**
- [ ] **Configure CORS** อย่างถูกต้อง

### เดือนนี้

- [ ] **Implement API gateway** (AWS API Gateway, Kong, etc.)
- [ ] **Setup API monitoring** + alerting
- [ ] **Penetration test** API
- [ ] **API versioning strategy**
- [ ] **Token expiration policy**

### สำหรับ Tech Lead

- [ ] **API security policy** เอกสาร
- [ ] **Code review checklist** สำหรับ API endpoints
- [ ] **Training ทีม** เรื่อง OWASP API Top 10
- [ ] **Quarterly API security audit**

---

## บทเรียนชีวิตจากบทความนี้

> **ให้สิทธิ์เท่าที่จำเป็น — ทั้งในระบบและในชีวิต**

หลักการ **least privilege** เป็นกฎทองที่ใช้ได้ทุกที่ในชีวิต:

#### ในการเงิน

- **บัตรเครดิต**: ใช้ใบที่ limit ต่ำสำหรับ online — ไม่ใช่ใบ premium ที่ limit สูง
- **Transfer money**: เปิด daily limit ที่ต่ำ — กัน ATM scam
- **Joint account**: only joint สำหรับเงินที่ใช้ร่วม — ไม่ใช่บัญชีออมทรัพย์ทั้งหมด

#### ในการทำงาน

- **Permissions ใน Google Drive**: share เฉพาะคนที่ต้องการ — ไม่ใช่ "anyone with link"
- **Code repo access**: dev ที่อยู่ project A ไม่ต้องเห็น project B
- **Production access**: dev ดู logs ได้ — แต่ไม่ต้อง shell access

#### ในความสัมพันธ์

- **เปิดเผยข้อมูลส่วนตัว**: ไม่ทุกคนต้องรู้ทุกอย่างของคุณ
- **ขอความช่วยเหลือ**: เลือกคนที่เกี่ยวข้องเฉพาะ — ไม่ใช่ broadcast ใน Facebook
- **ความสนิท**: เพื่อนต่างระดับ — ไม่ทุกคนเป็น "best friend"

#### ในการเป็น parent

- **มือถือลูก**: parental control + restrictions ที่จำเป็น
- **บัญชี internet**: kids account, time limits
- **Money allowance**: เพิ่มตามอายุ + ความรับผิดชอบ

> **คนที่ "ให้สิทธิ์เกินจำเป็น" — ไม่ใช่คนใจดี แต่เป็นคน naive**

ในประเด็น API:

- ทุก endpoint ที่ไม่จำเป็น = attack surface
- ทุก permission ที่เกิน = risk
- ทุก field ใน response ที่ไม่ใช้ = data exposure
- ทุก rate limit ที่กว้าง = abuse opportunity

**Default to deny** — เปิดเฉพาะที่จำเป็น

นี่ใช้ได้กับเรื่อง:

- **Software design**: deny-by-default permissions
- **Network**: deny-all firewall rules + explicit allow
- **Cloud IAM**: minimum role assignment
- **Frontend**: hide UI สำหรับ feature ที่ user ไม่มีสิทธิ์
- **Database**: least-privilege user
- **Life**: ระมัดระวังก่อนเปิดเผย

> **คุณภาพของระบบ — เห็นจาก default behavior ของมัน**  
> **ระบบที่ดี: default คือ secure, ต้องแก้ให้ open**  
> **ระบบที่แย่: default คือ open, ต้องแก้ให้ secure**

ในชีวิตคนเราก็เช่นกัน — การ **trust by default** มี cost  
การ **verify by default** ดูเหมือนเหนื่อย แต่ปลอดภัยกว่ามากในระยะยาว

นั่นคือบทเรียนของบทความนี้ — และของชีวิตในเวลาเดียวกันครับ

ในบทถัดไป **Cloud Security** — เพราะปัจจุบันเกือบทุก API + service อยู่บน cloud — ความเสี่ยงตามไปด้วย

---

## อภิธานศัพท์ (Glossary)

- **API (Application Programming Interface):** interface ที่ programs สื่อสารกัน
- **REST API:** API style ที่ใช้ HTTP + resources
- **GraphQL:** API query language alternative ของ REST
- **Authentication (AuthN):** ตรวจว่าใคร
- **Authorization (AuthZ):** ตรวจว่าทำได้ไหม
- **OAuth 2.0:** standard สำหรับ delegated authorization
- **OpenID Connect (OIDC):** identity layer บน OAuth
- **JWT (JSON Web Token):** self-contained token ที่ sign ด้วย key
- **Access Token:** token ที่ใช้ access API (มักหมดอายุเร็ว)
- **Refresh Token:** token ที่ใช้แลก access token ใหม่
- **PKCE (Proof Key for Code Exchange):** mechanism ป้องกัน auth code interception
- **API Key:** static credential สำหรับ identification
- **Rate Limiting:** จำกัดจำนวน request ต่อเวลา
- **Throttling:** ชะลอ request ที่เกิน
- **CORS (Cross-Origin Resource Sharing):** browser feature บล็อก cross-origin requests
- **Preflight Request:** OPTIONS request ที่ browser ส่งก่อน complex requests
- **BOLA (Broken Object Level Authorization):** ขาดการเช็ค ownership ของ object
- **IDOR (Insecure Direct Object Reference):** = BOLA ในศัพท์เก่า
- **Mass Assignment:** อนุญาตให้ user set ทุก field ผ่าน API
- **OpenAPI / Swagger:** มาตรฐานเอกสาร API
- **API Gateway:** middleware ที่จัดการ auth, rate limit, routing สำหรับ APIs
- **Sunset Header:** HTTP header ที่บอกวันหมดอายุของ endpoint
- **Deprecation Header:** HTTP header ที่บอกว่า endpoint deprecated

---

## สรุป

1. **Authentication ≠ Authorization** — ต้องเช็คทั้งสองทุก endpoint
2. **OAuth 2.0 / OIDC** เป็นมาตรฐานสำหรับ delegated auth — ใช้แทนการ implement เอง
3. **API keys ต้อง scoped + revocable + audited** — ไม่ใช่ super-key ใบเดียว
4. **Rate limiting** เป็น mandatory — ไม่ใช่ optional
5. **CORS เป็น browser feature** — ไม่ใช่ authorization, server ยังต้องเช็ค auth
6. **OWASP API Top 10** — รู้ทุกข้อ + audit project
7. **BOLA (IDOR) เป็น vulnerability ที่พบบ่อยที่สุด** — ทุก endpoint ที่มี ID ต้อง verify ownership
8. **Mass assignment / excessive data exposure** — ใช้ allow-list ของ fields
9. **API documentation** ที่ดี = security feature
10. **Least privilege everywhere** — default deny, explicit allow

ในบทถัดไป — **Cloud Security** — Infrastructure อยู่บน AWS/GCP/Azure ความเสี่ยงตามไปด้วย พร้อมความเสี่ยง specific ของ cloud architecture

— Claude Opus 4.6
