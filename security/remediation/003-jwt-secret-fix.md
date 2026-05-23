# Remediation: Hardcoded JWT Secret

**Finding ID:** 003
**Severity:** Critical
**CWE:** CWE-798
**File:** `lib/insecurity.ts`

## Root Cause

JWT signing secret is a static hardcoded string committed to version control.
Anyone with repo access (or access to the public GitHub history) can forge tokens.

## Fix

```typescript
// BEFORE (vulnerable)
const jwtSecret = 'your_jwt_secret'

// AFTER (safe)
const jwtSecret = process.env.JWT_SECRET
if (!jwtSecret || jwtSecret.length < 32) {
  throw new Error('JWT_SECRET must be set and at least 32 characters long')
}
```

Store the secret in GitHub Actions Secrets:
1. Repo → Settings → Secrets and variables → Actions → New repository secret
2. Name: `JWT_SECRET`, Value: `<generated 64-char random hex>`

Reference in workflow:
```yaml
env:
  JWT_SECRET: ${{ secrets.JWT_SECRET }}
```

## Generating a Strong Secret

```bash
openssl rand -hex 64
```

## Verification

- Confirm `grep -r "your_jwt_secret" .` returns no results
- Confirm app starts and token signing works with env var set
- Confirm app throws on startup if env var is missing
