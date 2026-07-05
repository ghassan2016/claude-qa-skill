# 08 — Security Testing & InfoSec

---

## Security Testing Philosophy

Security testing is **not optional** and **not a final step** — it is embedded in every phase.  
Every input is a potential attack vector. Every endpoint is a potential exposure.  
Every session token is a target.

---

## OWASP Top 10 — Testing Checklist (2021)

### A01 — Broken Access Control ⚠️ Most Critical
**What to test:**
- [ ] Authenticated user can access ONLY their own data
- [ ] User role boundaries enforced server-side (not just UI)
- [ ] Admin routes inaccessible to regular users
- [ ] IDOR (Insecure Direct Object Reference) — can user access `/api/orders/123` when they own order 456?
- [ ] Horizontal privilege escalation — can User A access User B's account?
- [ ] Vertical privilege escalation — can User escalate to Admin?
- [ ] JWT `role` claim not trusted blindly — verified against database
- [ ] Directory traversal not possible on file access endpoints

**Test Procedure:**
```
1. Log in as User A — note all resource IDs (user ID, order ID, etc.)
2. Log in as User B — attempt to access User A's resources by ID
3. Expected: 403 Forbidden
4. Log in as regular user — attempt admin endpoints directly
5. Expected: 403 Forbidden
```

---

### A02 — Cryptographic Failures
**What to test:**
- [ ] Passwords stored as bcrypt/argon2 hash (never plaintext, never MD5/SHA1)
- [ ] Sensitive data (PII, tokens) encrypted at rest
- [ ] All traffic over HTTPS — HTTP redirects to HTTPS
- [ ] TLS 1.2+ only (1.0/1.1 disabled)
- [ ] Sensitive data not in URL parameters (visible in logs/history)
- [ ] API responses don't include sensitive fields unnecessarily (password hash, internal IDs)
- [ ] JWT signed with strong key (HS256 with long secret or RS256)
- [ ] JWT `alg: none` attack — server must reject unsigned tokens

**Test Procedure:**
```
1. Attempt login — check network response for any password data
2. Check response headers: Strict-Transport-Security present?
3. Send request to HTTP version — does it redirect to HTTPS?
4. Decode JWT — does payload contain sensitive data?
5. Modify JWT alg to "none" — does server accept? (MUST NOT)
```

---

### A03 — Injection
**What to test:**
- [ ] SQL Injection on all user inputs that query database
- [ ] NoSQL Injection on MongoDB/Firestore queries
- [ ] XSS (Cross-Site Scripting) on all displayed user input
- [ ] Command injection on any file operation endpoints
- [ ] SSRF (Server-Side Request Forgery) on URL input fields

**SQL Injection Test Inputs:**
```
' OR '1'='1
'; DROP TABLE users; --
1; SELECT * FROM users
' UNION SELECT null, username, password FROM users --
```

**XSS Test Inputs:**
```
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
"><script>alert('XSS')</script>
javascript:alert('XSS')
<svg onload=alert('XSS')>
```

**Expected Result:** All inputs sanitized — no script executes, no SQL executes.

---

### A04 — Insecure Design
**What to test:**
- [ ] Password reset flow — token expires (< 15 minutes), single-use
- [ ] Account enumeration — login error should NOT say "email not found" vs "wrong password"
- [ ] Rate limiting on login, registration, password reset
- [ ] Multi-step transactions can't be manipulated by skipping steps
- [ ] Business logic — price cannot be modified client-side before payment

---

### A05 — Security Misconfiguration
**What to test:**
- [ ] Error pages don't expose stack traces or version numbers
- [ ] Directory listing disabled on web server
- [ ] Default credentials changed (admin/admin, admin/password)
- [ ] Unnecessary features/services disabled
- [ ] Security headers present (see header checklist below)
- [ ] CORS restricted to known origins (not `*`)
- [ ] Debug mode disabled in production
- [ ] `.env` file not accessible via browser

**Test .env exposure:**
```
GET /.env
GET /.env.local
GET /.env.production
Expected: 404 or 403
```

---

### A06 — Vulnerable and Outdated Components
**What to test:**
- [ ] `npm audit` / `yarn audit` run — zero Critical/High vulnerabilities in dependencies
- [ ] No dependency pinned to a version with a known CVE
- [ ] Framework versions (Next.js, Express, Flutter) up to date or on LTS
- [ ] Auth libraries (Clerk, NextAuth, Supabase Auth) on latest patch
- [ ] Docker base images scanned (if applicable)
- [ ] No unused/abandoned packages in `package.json` / `pubspec.yaml`
- [ ] `node_modules` not committed to git

**Test Procedure:**
```bash
# Node.js
npm audit --audit-level=high

# Check for outdated packages
npm outdated

# Quick CVE check
npx better-npm-audit audit
```

**Expected:** Zero High/Critical advisories. Medium — review and document risk.

---

### A07 — Authentication & Session Failures
**What to test:**
- [ ] Brute force protection on login (lockout after 5-10 attempts)
- [ ] Session token invalidated on logout (server-side)
- [ ] Token not reusable after expiry
- [ ] Session fixation — token rotated after successful login
- [ ] "Remember me" token has reasonable expiry and can be revoked
- [ ] Concurrent sessions handled (user logged out elsewhere)
- [ ] MFA bypass not possible (if MFA implemented)
- [ ] OAuth state parameter validated (CSRF protection)

**Test Procedure:**
```
1. Log in → copy session token
2. Log out
3. Replay old token in Authorization header
4. Expected: 401 Unauthorized
```

---

### A08 — Software and Data Integrity Failures
**What to test:**
- [ ] CI/CD pipeline does not allow untrusted code to run with elevated permissions
- [ ] npm scripts in `package.json` don't execute arbitrary remote scripts
- [ ] No `curl | bash` patterns in setup or deploy scripts
- [ ] Dependency integrity checked — `package-lock.json` or `yarn.lock` committed and reviewed
- [ ] No unsigned or unverified packages installed (check `npm install` output)
- [ ] Deserialization of untrusted data handled safely (no `eval()`, no `Function()` constructor)
- [ ] Webhooks validate incoming payload signatures (e.g. Stripe `stripe.webhooks.constructEvent`, Clerk `svix` verification)
- [ ] Auto-update mechanisms (if any) verify integrity before applying

**Test Procedure:**
```
1. Check CI/CD config — does it pull from unverified sources?
2. Review package.json scripts for remote execution patterns
3. Verify webhook handlers check signatures before processing
4. Check for eval() or dynamic code execution with user input
Expected: No unsigned execution. All webhook payloads verified.
```

---

### A09 — Logging & Monitoring Failures
**What to test:**
- [ ] Failed login attempts logged with timestamp and IP
- [ ] Auth events logged (login, logout, password change)
- [ ] Access to sensitive resources logged
- [ ] Logs do NOT contain passwords or tokens
- [ ] Monitoring alert configured for unusual activity
- [ ] Log tampering not possible by application user

---

## Authentication Security — Detailed

### JWT Testing
| Test | Expected |
|------|---------|
| Expired token | 401 Unauthorized |
| Tampered payload | 401 Unauthorized |
| `alg: none` attack | 401 Unauthorized |
| Wrong signature | 401 Unauthorized |
| Token from another user | 403 Forbidden |
| Missing Authorization header | 401 Unauthorized |

### Session Management
- [ ] Token has appropriate expiry (access: 15-60min, refresh: 7-30 days)
- [ ] Refresh token is single-use (rotated on each use)
- [ ] Refresh token invalidated on logout
- [ ] Refresh token stored in `httpOnly` cookie (not localStorage)
- [ ] Access token not stored in localStorage (XSS risk) — prefer memory or `httpOnly` cookie

### Password Security
- [ ] Minimum length ≥ 8 characters enforced
- [ ] Strength requirement (not just length)
- [ ] Known/breached passwords rejected (HaveIBeenPwned check)
- [ ] No maximum length limit (prevents proper hashing)
- [ ] Password not shown in clear text in any API response or log
- [ ] Change password requires current password confirmation

---

## API Security — Detailed

### Endpoint Authorization Testing Matrix
For each endpoint, test with all roles:

| Role | Access Level |
|------|-------------|
| Unauthenticated | Public endpoints only |
| Regular User | Own resources only |
| Admin | All resources |

```
Test each protected endpoint with:
1. No token → expect 401
2. Expired token → expect 401
3. Another user's valid token → expect 403
4. Admin token on user endpoint → expect 200
5. User token on admin endpoint → expect 403
```

### Rate Limiting Validation
- [ ] Login endpoint: max 5-10 attempts per minute per IP
- [ ] Registration: max 3 per hour per IP
- [ ] Password reset: max 3 per hour per email
- [ ] API endpoints: reasonable RPM limits per user
- [ ] Response on rate limit: 429 with `Retry-After` header

### Input Validation on API
- [ ] Required fields validated server-side (not just client)
- [ ] String length limits enforced
- [ ] Numeric ranges validated
- [ ] Enum values validated (not accepting arbitrary strings)
- [ ] File type and size validated on upload
- [ ] Malicious file upload rejected (PHP, EXE, scripts)

### CORS Configuration
```
Secure:
  Access-Control-Allow-Origin: https://yourdomain.com

Insecure (must flag):
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Credentials: true  (with wildcard = critical vulnerability)
```

---

## Frontend Security — Detailed

### XSS Prevention
- [ ] User-generated content sanitized before display
- [ ] `dangerouslySetInnerHTML` (React) / `[innerHTML]` (Angular) never used without `DOMPurify`
- [ ] `Content-Security-Policy` header configured
- [ ] No inline `<script>` tags with user data
- [ ] URL parameters not directly injected into DOM

### Sensitive Data Exposure
- [ ] No API keys, secrets, or tokens in frontend bundle
- [ ] No PII in URL params (visible in server logs, browser history)
- [ ] Sensitive data not logged to console in production
- [ ] LocalStorage only for non-sensitive, non-auth data

### Clickjacking
- [ ] `X-Frame-Options: DENY` or `SAMEORIGIN` header present
- [ ] CSP `frame-ancestors` directive configured

---

## Security Headers Checklist

Test using: https://securityheaders.com or DevTools → Network → Response Headers

| Header | Required Value | Risk if Missing |
|--------|--------------|----------------|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | HTTPS downgrade attacks |
| `Content-Security-Policy` | Defined policy | XSS attacks |
| `X-Content-Type-Options` | `nosniff` | MIME sniffing attacks |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Clickjacking |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Data leakage in Referer |
| `Permissions-Policy` | Restrict camera/mic/geo as needed | Feature abuse |
| `Cache-Control` | `no-store` on sensitive pages | Sensitive data cached |

---

## Infrastructure Security Basics

### Environment & Secrets
- [ ] `.env` not committed to git (`.gitignore` verified)
- [ ] Production secrets different from staging
- [ ] No secrets hardcoded in source code
- [ ] Secrets manager used for production (AWS Secrets Manager, Doppler, etc.)

### File Storage
- [ ] S3 / Cloud storage buckets NOT publicly accessible
- [ ] Uploaded files served via signed URLs with expiry
- [ ] No direct user access to file system paths
- [ ] Uploaded files scanned for malware (if handling user uploads in enterprise context)

### Error Handling
- [ ] Production errors return generic message to user
- [ ] Full stack trace logged server-side only (not in response)
- [ ] Database error messages never exposed to client
- [ ] API version or framework version not in response headers

---

## Security Audit Output Format

```markdown
## Security Assessment — [Feature / System / Release]
**Assessor:** [OWNER_NAME]  
**Date:** [YYYY-MM-DD]  
**Scope:** [What was tested]  
**Methodology:** OWASP Top 10 + Custom checks

---

### Overall Risk Level
🔴 CRITICAL / 🟠 HIGH / 🟡 MEDIUM / 🟢 LOW

---

### Findings

#### 🔴 Critical (fix before any release)
**SEC-001 — [Title]**
- OWASP Reference: A[XX]
- Description: [What the vulnerability is]
- Proof of Concept: [How to reproduce]
- Impact: [What attacker can do]
- Affected: [Endpoint/Component]
- Remediation: [Specific fix with code/config example]

#### 🟠 High (fix before next release)
**SEC-002 — [Title]**
[Same format]

#### 🟡 Medium (address within sprint)
**SEC-003 — [Title]**
[Same format]

#### 🟢 Low (track and schedule)
**SEC-004 — [Title]**
[Same format]

---

### Security Posture Summary
| Domain | Status | Notes |
|--------|--------|-------|
| Authentication | ✅ / ⚠️ / ❌ | |
| Authorization | ✅ / ⚠️ / ❌ | |
| Input Validation | ✅ / ⚠️ / ❌ | |
| Data Exposure | ✅ / ⚠️ / ❌ | |
| Security Headers | ✅ / ⚠️ / ❌ | |
| Session Management | ✅ / ⚠️ / ❌ | |

### Recommendation
[GO / NO-GO for release from security perspective]
```
