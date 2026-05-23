# Security Findings — OWASP Juice Shop

**Scan Date:** 2026-05-22
**Target:** OWASP Juice Shop (intentionally vulnerable Node.js application)
**Pipeline:** GitHub Actions — Semgrep (SAST) + Trivy (SCA/Container) + OWASP ZAP (DAST)

---

## Summary

| Severity | SAST (Semgrep) | SCA (Trivy) | DAST (ZAP) | Total |
|----------|---------------|-------------|------------|-------|
| Critical | 1 | 3 | 1 | 5 |
| High     | 2 | 8 | 3 | 13 |
| Medium   | 4 | 5 | 6 | 15 |
| Low      | 2 | 2 | 4 | 8 |

---

## Finding 001 — SQL Injection via Login Endpoint

**Tool:** SAST (Semgrep)
**Severity:** Critical
**Rule:** `javascript.sequelize.security.audit.sequelize-injection-from-request`
**File:** `routes/login.ts`
**CWE:** CWE-89 — Improper Neutralization of Special Elements used in SQL Command
**OWASP Top 10:** A03:2021 — Injection

### Description

The login endpoint constructs a Sequelize query using raw user-supplied input without parameterization. An attacker can bypass authentication by submitting `' OR 1=1--` as the email field.

### Evidence

```javascript
// routes/login.ts (vulnerable)
models.sequelize.query(
  `SELECT * FROM Users WHERE email = '${req.body.email}' AND password = '${req.body.password}'`
)
```

### Remediation

Use Sequelize's parameterized query API or ORM methods instead of raw string interpolation:

```javascript
// Fixed: use parameterized query
models.User.findOne({
  where: {
    email: req.body.email,
    password: hash(req.body.password)
  }
})
```

**Status:** Documented (intentional in Juice Shop — see `remediation/001-sqli-fix.md`)

---

## Finding 002 — Reflected Cross-Site Scripting (XSS)

**Tool:** DAST (OWASP ZAP)
**Severity:** High
**ZAP Alert ID:** 40012
**Endpoint:** `GET /rest/products/search?q=<payload>`
**CWE:** CWE-79 — Improper Neutralization of Input During Web Page Generation
**OWASP Top 10:** A03:2021 — Injection

### Description

The search endpoint reflects user input directly into the response without encoding. Payloads such as `<iframe src="javascript:alert('xss')">` execute in the victim's browser.

### Evidence

```
Request:  GET /rest/products/search?q=<iframe src="javascript:alert('xss')">
Response: 200 OK — payload reflected unencoded in JSON response consumed by Angular
```

### Remediation

- Sanitize output with DOMPurify or Angular's built-in sanitization
- Set `Content-Security-Policy: script-src 'self'` response header
- Validate and reject input containing HTML/script tags at the API layer

**Status:** Documented

---

## Finding 003 — Hardcoded JWT Secret

**Tool:** SAST (Semgrep)
**Severity:** Critical
**Rule:** `javascript.hardcoded-secret`
**File:** `lib/insecurity.ts`
**CWE:** CWE-798 — Use of Hard-coded Credentials
**OWASP Top 10:** A02:2021 — Cryptographic Failures

### Description

The application signs JWT tokens using a hardcoded secret (`"your_jwt_secret"`). An attacker who knows this secret can forge valid tokens for any user, including administrators.

### Evidence

```javascript
// lib/insecurity.ts
const jwtSecret = 'your_jwt_secret'
```

### Remediation

```javascript
// Load secret from environment variable, never commit it
const jwtSecret = process.env.JWT_SECRET
if (!jwtSecret) throw new Error('JWT_SECRET environment variable is required')
```

Store the secret in a secrets manager (AWS Secrets Manager, GitHub Secrets, Vault).

**Status:** Documented (see `remediation/003-jwt-secret-fix.md`)

---

## Finding 004 — Broken Access Control (IDOR)

**Tool:** Manual verification after DAST flag
**Severity:** High
**Endpoint:** `GET /rest/users/:id`
**CWE:** CWE-284 — Improper Access Control
**OWASP Top 10:** A01:2021 — Broken Access Control

### Description

Authenticated users can retrieve the profile data of any other user by changing the numeric `id` parameter in the request. There is no server-side check verifying the requesting user owns the resource.

### Evidence

```
Authenticated as user ID 1:
GET /rest/users/2  → 200 OK (returns user 2's email, password hash, etc.)
GET /rest/users/3  → 200 OK (returns user 3's data)
```

### Remediation

```javascript
// Add ownership check in the route handler
router.get('/users/:id', security.isAuthorized, (req, res) => {
  if (req.params.id !== req.user.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Access denied' })
  }
  // ... proceed
})
```

**Status:** Documented

---

## Finding 005 — Outdated Dependencies with Known CVEs

**Tool:** Trivy (SCA — filesystem scan)
**Severity:** Critical / High (multiple)
**OWASP Top 10:** A06:2021 — Vulnerable and Outdated Components

### Selected Critical/High CVEs

| Package | Version | CVE | Severity | Description |
|---------|---------|-----|----------|-------------|
| `jsonwebtoken` | < 9.0.0 | CVE-2022-23529 | High | Unrestricted key injection via `algorithms` option |
| `express` | < 4.18.2 | CVE-2022-24999 | High | Open redirect via malformed URL |
| `sanitize-html` | < 2.7.1 | CVE-2022-25887 | Critical | ReDoS via crafted regex input |
| `node-ipc` | 10.1.1 | CVE-2022-23812 | Critical | Protestware — intentional data destruction |

### Remediation

```bash
# Update vulnerable packages
npm audit fix

# For packages requiring major version bumps
npm install jsonwebtoken@^9.0.0
npm install express@^4.18.2
npm install sanitize-html@^2.7.1

# Verify no remaining high/critical issues
npm audit --audit-level=high
```

**Status:** Documented (intentional in Juice Shop for demonstration purposes)

---

## Finding 006 — Missing Security Headers

**Tool:** DAST (OWASP ZAP)
**Severity:** Medium
**Affected Headers:** `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`, `Strict-Transport-Security`
**CWE:** CWE-693 — Protection Mechanism Failure
**OWASP Top 10:** A05:2021 — Security Misconfiguration

### Description

The application does not set standard HTTP security headers, leaving it vulnerable to clickjacking, MIME-type sniffing, and protocol downgrade attacks.

### Remediation

Add Helmet.js middleware to the Express app:

```javascript
const helmet = require('helmet')

app.use(helmet())
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", "data:"]
  }
}))
```

**Status:** Documented (see `remediation/006-security-headers-fix.md`)

---

## Remediation Tracking

| Finding | ID | Severity | Status |
|---------|----|----------|--------|
| SQL Injection | 001 | Critical | Documented |
| Reflected XSS | 002 | High | Documented |
| Hardcoded JWT Secret | 003 | Critical | Documented |
| IDOR / Broken Access Control | 004 | High | Documented |
| Vulnerable Dependencies | 005 | Critical/High | Documented |
| Missing Security Headers | 006 | Medium | Documented |
