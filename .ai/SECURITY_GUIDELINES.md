# Security Guidelines

## Overview

This document outlines security best practices for developing secure, production-ready applications. All AI agents and developers MUST follow these guidelines.

---

## OWASP Top 10 Prevention

### 1. Injection (A03:2021)

#### SQL Injection Prevention

```python
# NEVER do this
query = f"SELECT * FROM users WHERE id = {user_id}"

# ALWAYS use parameterized queries (SQLAlchemy handles this)
user = await db.execute(select(User).where(User.id == user_id))

# For raw queries, use parameters
result = await db.execute(
    text("SELECT * FROM users WHERE email = :email"),
    {"email": email}
)
```

#### Command Injection Prevention

```python
# NEVER do this
os.system(f"convert {filename} output.png")

# Use subprocess with list arguments
import subprocess
subprocess.run(["convert", filename, "output.png"], check=True)

# Better: Use libraries instead of shell commands
from PIL import Image
img = Image.open(filename)
img.save("output.png")
```

### 2. Broken Authentication (A07:2021)

#### Password Requirements

```python
# Minimum password requirements
PASSWORD_MIN_LENGTH = 12
PASSWORD_REQUIRE_UPPERCASE = True
PASSWORD_REQUIRE_LOWERCASE = True
PASSWORD_REQUIRE_DIGIT = True
PASSWORD_REQUIRE_SPECIAL = True

# Use bcrypt with sufficient rounds
from passlib.context import CryptContext
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto", bcrypt__rounds=12)
```

#### JWT Security

```python
# JWT configuration
JWT_ALGORITHM = "HS256"
JWT_ACCESS_TOKEN_EXPIRE_MINUTES = 15  # Short-lived access tokens
JWT_REFRESH_TOKEN_EXPIRE_DAYS = 7

# Include essential claims
payload = {
    "sub": str(user_id),
    "exp": datetime.utcnow() + timedelta(minutes=JWT_ACCESS_TOKEN_EXPIRE_MINUTES),
    "iat": datetime.utcnow(),
    "jti": str(uuid4()),  # Unique token ID for revocation
    "type": "access"
}
```

#### Session Security

```python
# Session configuration
SESSION_COOKIE_SECURE = True      # HTTPS only
SESSION_COOKIE_HTTPONLY = True    # No JavaScript access
SESSION_COOKIE_SAMESITE = "Lax"   # CSRF protection
SESSION_COOKIE_NAME = "__Host-session"  # Secure prefix
```

### 3. Sensitive Data Exposure (A02:2021)

#### Environment Variables

```python
# NEVER hardcode secrets
DATABASE_URL = os.getenv("DATABASE_URL")
JWT_SECRET = os.getenv("JWT_SECRET")

# Validate required env vars on startup
REQUIRED_ENV_VARS = ["DATABASE_URL", "JWT_SECRET", "ENCRYPTION_KEY"]

def validate_env():
    missing = [var for var in REQUIRED_ENV_VARS if not os.getenv(var)]
    if missing:
        raise ValueError(f"Missing required environment variables: {missing}")
```

#### Data Encryption

```python
# Encrypt sensitive data at rest
from cryptography.fernet import Fernet

def encrypt_sensitive_data(data: str, key: bytes) -> str:
    f = Fernet(key)
    return f.encrypt(data.encode()).decode()

def decrypt_sensitive_data(encrypted: str, key: bytes) -> str:
    f = Fernet(key)
    return f.decrypt(encrypted.encode()).decode()
```

### 4. XML External Entities (A05:2021)

```python
# Disable external entity processing
import defusedxml.ElementTree as ET

# NEVER use standard xml library for untrusted input
# from xml.etree import ElementTree  # UNSAFE

tree = ET.parse(xml_file)  # Safe
```

### 5. Broken Access Control (A01:2021)

#### Authorization Checks

```python
# Always verify ownership/permissions
@router.get("/exams/{exam_id}")
async def get_exam(
    exam_id: int,
    current_user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db)
):
    exam = await ExamService(db).get_by_id(exam_id)
    if not exam:
        raise HTTPException(status_code=404)

    # Check access permission
    if exam.created_by_id != current_user.id and not current_user.is_admin:
        raise HTTPException(status_code=403, detail="Access denied")

    return exam
```

#### Role-Based Access Control (RBAC)

```python
from enum import Enum
from functools import wraps

class Role(str, Enum):
    STUDENT = "student"
    TEACHER = "teacher"
    ADMIN = "admin"

def require_role(*roles: Role):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, current_user: User = Depends(get_current_user), **kwargs):
            if current_user.role not in roles:
                raise HTTPException(
                    status_code=403,
                    detail=f"Role {current_user.role} not authorized"
                )
            return await func(*args, current_user=current_user, **kwargs)
        return wrapper
    return decorator

@router.delete("/users/{user_id}")
@require_role(Role.ADMIN)
async def delete_user(user_id: int, current_user: User = Depends(get_current_user)):
    ...
```

### 6. Security Misconfiguration (A05:2021)

#### CORS Configuration

```python
# Production CORS settings
from fastapi.middleware.cors import CORSMiddleware

ALLOWED_ORIGINS = os.getenv("ALLOWED_ORIGINS", "").split(",")

app.add_middleware(
    CORSMiddleware,
    allow_origins=ALLOWED_ORIGINS,  # Never use ["*"] in production
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH"],
    allow_headers=["Authorization", "Content-Type"],
    max_age=86400,  # Cache preflight for 24 hours
)
```

#### Security Headers

```python
from fastapi import FastAPI
from starlette.middleware.base import BaseHTTPMiddleware

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request, call_next):
        response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Content-Security-Policy"] = "default-src 'self'"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        response.headers["Permissions-Policy"] = "geolocation=(), microphone=(), camera=()"
        return response

app.add_middleware(SecurityHeadersMiddleware)
```

### 7. Cross-Site Scripting (XSS) (A03:2021)

#### Backend Output Encoding

```python
import html

def safe_output(user_input: str) -> str:
    return html.escape(user_input)

# For JSON responses, FastAPI handles encoding automatically
# But be careful with raw HTML responses
```

#### Frontend XSS Prevention

```tsx
// React automatically escapes JSX
<div>{userInput}</div>  // Safe - escaped

// NEVER use dangerouslySetInnerHTML with user input
<div dangerouslySetInnerHTML={{ __html: userInput }} />  // DANGEROUS

// If HTML is needed, use a sanitizer
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

#### Content Security Policy

```python
# Strict CSP header
CSP_POLICY = "; ".join([
    "default-src 'self'",
    "script-src 'self'",
    "style-src 'self' 'unsafe-inline'",  # Required for some CSS-in-JS
    "img-src 'self' data: https:",
    "font-src 'self'",
    "connect-src 'self' https://api.example.com",
    "frame-ancestors 'none'",
    "base-uri 'self'",
    "form-action 'self'"
])
```

### 8. Insecure Deserialization (A08:2021)

```python
# NEVER use pickle with untrusted data
# import pickle  # DANGEROUS for untrusted input

# Use JSON for data serialization
import json
data = json.loads(user_input)

# Validate with Pydantic
from pydantic import BaseModel, validator

class UserInput(BaseModel):
    name: str
    age: int

    @validator('age')
    def validate_age(cls, v):
        if v < 0 or v > 150:
            raise ValueError('Invalid age')
        return v
```

### 9. Using Components with Known Vulnerabilities (A06:2021)

- Run `pnpm audit` and `pip-audit` regularly
- Enable Dependabot alerts
- Update dependencies promptly
- Check CVE databases before adding new dependencies

### 10. Insufficient Logging & Monitoring (A09:2021)

```python
import logging
from datetime import datetime

logger = logging.getLogger(__name__)

# Log security events
def log_security_event(event_type: str, user_id: int, details: dict):
    logger.warning(
        f"SECURITY_EVENT | type={event_type} | user={user_id} | "
        f"timestamp={datetime.utcnow().isoformat()} | details={details}"
    )

# Events to log:
# - Failed login attempts
# - Password changes
# - Permission changes
# - Access denied events
# - Data exports
# - Admin actions
```

---

## Rate Limiting

### Backend Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

# Apply rate limits
@router.post("/auth/login")
@limiter.limit("5/minute")  # 5 attempts per minute
async def login(request: Request, credentials: LoginCredentials):
    ...

@router.post("/auth/register")
@limiter.limit("3/hour")  # 3 registrations per hour per IP
async def register(request: Request, user_data: UserCreate):
    ...

# API rate limits
@router.get("/api/exams")
@limiter.limit("100/minute")  # 100 requests per minute
async def list_exams(request: Request):
    ...
```

### Rate Limit Headers

```python
# Include rate limit info in responses
response.headers["X-RateLimit-Limit"] = "100"
response.headers["X-RateLimit-Remaining"] = "95"
response.headers["X-RateLimit-Reset"] = "1640000000"
```

---

## Input Validation

### Backend Validation

```python
from pydantic import BaseModel, Field, EmailStr, validator
import re

class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(..., min_length=12, max_length=128)
    username: str = Field(..., min_length=3, max_length=50, pattern=r"^[a-zA-Z0-9_]+$")

    @validator('password')
    def validate_password(cls, v):
        if not re.search(r'[A-Z]', v):
            raise ValueError('Password must contain uppercase letter')
        if not re.search(r'[a-z]', v):
            raise ValueError('Password must contain lowercase letter')
        if not re.search(r'\d', v):
            raise ValueError('Password must contain digit')
        if not re.search(r'[!@#$%^&*(),.?":{}|<>]', v):
            raise ValueError('Password must contain special character')
        return v
```

### Frontend Validation

```typescript
import { z } from 'zod';

const userSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z
    .string()
    .min(12, 'Password must be at least 12 characters')
    .regex(/[A-Z]/, 'Must contain uppercase')
    .regex(/[a-z]/, 'Must contain lowercase')
    .regex(/\d/, 'Must contain digit')
    .regex(/[!@#$%^&*]/, 'Must contain special character'),
  username: z
    .string()
    .min(3)
    .max(50)
    .regex(/^[a-zA-Z0-9_]+$/, 'Only alphanumeric and underscore'),
});
```

---

## File Upload Security

```python
import magic
from pathlib import Path
import uuid

ALLOWED_EXTENSIONS = {'.jpg', '.jpeg', '.png', '.gif', '.pdf'}
ALLOWED_MIME_TYPES = {'image/jpeg', 'image/png', 'image/gif', 'application/pdf'}
MAX_FILE_SIZE = 10 * 1024 * 1024  # 10MB

async def secure_file_upload(file: UploadFile) -> str:
    # Check file size
    content = await file.read()
    if len(content) > MAX_FILE_SIZE:
        raise HTTPException(400, "File too large")

    # Check extension
    ext = Path(file.filename).suffix.lower()
    if ext not in ALLOWED_EXTENSIONS:
        raise HTTPException(400, "Invalid file type")

    # Check MIME type (magic bytes)
    mime_type = magic.from_buffer(content, mime=True)
    if mime_type not in ALLOWED_MIME_TYPES:
        raise HTTPException(400, "Invalid file content")

    # Generate safe filename
    safe_filename = f"{uuid.uuid4()}{ext}"

    # Save outside web root
    upload_path = Path("/secure/uploads") / safe_filename
    upload_path.write_bytes(content)

    return safe_filename
```

---

## API Security Checklist

### Authentication
- [ ] Use HTTPS only
- [ ] Implement JWT with short expiration
- [ ] Use refresh token rotation
- [ ] Hash passwords with bcrypt (12+ rounds)
- [ ] Implement account lockout after failed attempts
- [ ] Use secure session cookies

### Authorization
- [ ] Verify permissions on every request
- [ ] Implement RBAC
- [ ] Check resource ownership
- [ ] Log access denied events

### Input/Output
- [ ] Validate all input with Pydantic/Zod
- [ ] Sanitize output to prevent XSS
- [ ] Use parameterized queries
- [ ] Limit request body size

### Infrastructure
- [ ] Enable CORS with specific origins
- [ ] Set security headers
- [ ] Implement rate limiting
- [ ] Use CSP headers

### Monitoring
- [ ] Log security events
- [ ] Monitor failed login attempts
- [ ] Set up alerts for anomalies
- [ ] Regular security audits

---

## Secrets Management

### Never Commit Secrets

```gitignore
# .gitignore
.env
.env.local
.env.*.local
*.pem
*.key
secrets/
```

### Environment Variable Template

```bash
# .env.example (commit this)
DATABASE_URL=postgresql://user:password@localhost:5432/db
JWT_SECRET=your-secret-key-here-min-32-chars
ENCRYPTION_KEY=your-encryption-key-here

# Generate secure secrets
# openssl rand -hex 32
```

### Secret Rotation

- Rotate secrets every 90 days
- Use different secrets per environment
- Implement secret versioning for zero-downtime rotation

---

## Security Testing

### Pre-Deployment Checklist

1. **Static Analysis**
   - Run Semgrep for security patterns
   - Run CodeQL analysis
   - Check for hardcoded secrets

2. **Dependency Audit**
   - `pnpm audit`
   - `pip-audit`
   - Check Dependabot alerts

3. **Container Scanning**
   - Scan Docker images with Trivy
   - Use minimal base images
   - Don't run as root

4. **Penetration Testing**
   - OWASP ZAP automated scan
   - Manual testing for business logic flaws

---

## Incident Response

### If a Security Incident Occurs

1. **Contain**: Isolate affected systems
2. **Assess**: Determine scope and impact
3. **Notify**: Inform stakeholders and users if data exposed
4. **Remediate**: Fix vulnerability, rotate compromised credentials
5. **Document**: Record incident and response
6. **Review**: Post-mortem and improve defenses

### Security Contacts

- Report vulnerabilities to: security@example.com
- Use responsible disclosure
- Do not publicly disclose until patched

---

*Last Updated: 2024-12-30*
