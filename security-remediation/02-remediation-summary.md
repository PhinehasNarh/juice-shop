# Remediation summary

This page collects the fixes into a few reusable actions, so you do not have to repeat the same work for every finding.

## 1. Application code patterns (CodeQL and Semgrep)

Most of the code findings fall into a small number of patterns. Fixing the pattern once teaches the fix for the whole group.

- Injection (SQL, Sequelize, code, template). Never build a command, query, or code string by joining user input. Use parameterized queries and safe APIs. Remove every use of eval.
- Path handling (path injection, zip slip, sendFile, file race). Reduce user input to a base name, resolve the final path, and confirm it stays inside the intended folder before you touch the file.
- Redirects and URL checks (open redirect, incomplete URL check, missing anchor). Parse URLs with the URL class and compare the hostname against an allow list. Do not check URLs with substring search.
- Output and logging (stack trace exposure, clear text logging, log injection, unquoted attribute, script tag). Return generic errors to clients, never log secrets, strip newlines from logged input, and always quote and encode values placed into HTML.
- Authentication and crypto (weak hash, insecure randomness, hardcoded secret, missing CSRF, user controlled bypass). Use bcrypt or argon2 for passwords, use the crypto module for tokens, keep secrets in environment variables, add CSRF protection, and make authorization decisions on the server.
- Availability (missing rate limiting, ReDoS, type confusion). Add a rate limiter to sensitive routes, cap input length before regular expressions, and validate the type of each parameter.

## 2. Dependency upgrades (Trivy)

The dependency findings are fixed by moving each package to a fixed version. Many are transitive, so the most reliable approach for a project like this is an `overrides` block in package.json. Add the block below, then run `npm install` and rerun the scan.

```json
{
  "overrides": {
    "@tootallnate/once": "^2.0.1",
    "base64url": "^3.0.0",
    "cookie": "^0.7.0",
    "crypto-js": "^4.2.0",
    "engine.io": "^6.6.4",
    "express-jwt": "^8.4.1",
    "file-type": "^21.3.1",
    "got": "^11.8.5",
    "http-cache-semantics": "^4.1.1",
    "js-yaml": "^4.2.0",
    "jsonwebtoken": "^9.0.2",
    "jws": "^4.0.0",
    "lodash": "^4.17.21",
    "messageformat": "^3.0.0",
    "minimatch": "^3.1.4",
    "moment": "^2.30.1",
    "multer": "^2.2.0",
    "sanitize-html": "^2.12.1",
    "socket.io": "^4.6.2",
    "socket.io-parser": "^4.2.6",
    "tar": "^7.5.16",
    "uuid": "^11.1.1",
    "ws": "^8.21.0"
  }
}
```

A few packages need more than a version bump.

- express-jwt. Moving from 0.1.3 to a current major changes the API, so the code that uses it must be updated as well.
- file-type. Recent majors are ESM only, which may require import changes.
- jsonwebtoken, crypto-js, sanitize-html. In Juice Shop these are intentionally old to support specific challenges. Upgrading them will fix the alerts but will also remove those challenges. Decide based on whether you want to keep the training value.

## 3. Secrets (Trivy)

- Move the RSA private key in lib/insecurity.ts to an environment variable or secret store and rotate it.
- Replace the JWT strings in the frontend test files with obvious placeholders or tokens built at test time.
- Add a secret scanner to the pre-commit hook so new secrets are blocked before they are committed.

## 4. Confirming the fixes

After applying any fix, push the change and let the DevSecOps workflow run again. CodeQL, Semgrep, and Trivy will re-scan, and any finding that is genuinely fixed will move to a fixed state automatically in the Security tab. Findings you accept as intentional can be dismissed with a reason, which is covered in the next page.
