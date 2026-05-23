# Remediation: SQL Injection — Login Endpoint

**Finding ID:** 001
**Severity:** Critical
**CWE:** CWE-89
**File:** `routes/login.ts`

## Root Cause

Raw string interpolation of user-controlled input directly into a SQL query.
Sequelize supports parameterized queries natively — this was not used.

## Fix

Replace raw query with Sequelize ORM method:

```typescript
// BEFORE (vulnerable)
models.sequelize.query(
  `SELECT * FROM Users WHERE email = '${req.body.email}' AND password = '${hash(req.body.password)}'`,
  { model: models.User, plain: true }
)

// AFTER (safe)
models.User.findOne({
  where: {
    email: req.body.email,
    password: hash(req.body.password)
  }
})
```

## Verification

Test with SQLi payloads — `' OR '1'='1`, `admin'--` — confirm they return 401 instead of 200.

Run automated regression:
```bash
npm test -- --grep "login"
```
