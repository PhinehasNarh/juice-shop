# Remediation: Missing HTTP Security Headers

**Finding ID:** 006
**Severity:** Medium
**CWE:** CWE-693
**File:** `app.ts` (Express app entry point)

## Root Cause

Helmet.js middleware is not configured, leaving the app without standard
defensive HTTP response headers.

## Fix

```bash
npm install helmet
```

```typescript
// app.ts — add near the top, before route registration
import helmet from 'helmet'

app.use(helmet())
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      frameAncestors: ["'none'"]
    }
  })
)
```

## Headers Added

| Header | Value | Protection |
|--------|-------|------------|
| `X-Content-Type-Options` | `nosniff` | MIME sniffing |
| `X-Frame-Options` | `DENY` | Clickjacking |
| `X-XSS-Protection` | `0` (disabled in favor of CSP) | Legacy XSS filter |
| `Strict-Transport-Security` | `max-age=31536000` | HTTPS downgrade |
| `Content-Security-Policy` | (see above) | XSS, injection |

## Verification

```bash
curl -I http://localhost:3000 | grep -E "X-Content|X-Frame|Strict|Content-Security"
```
