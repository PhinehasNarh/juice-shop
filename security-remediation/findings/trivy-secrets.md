# Trivy findings: hardcoded secrets

Trivy reported 7 hardcoded secret findings. A secret in source control is dangerous because anyone who can read the repository can read the secret. For a public repository, that means everyone.

### RSA private key committed in source

- Rule: `private-key`
- Severity: High
- Findings: 3

**What it is.** An RSA private key block is stored directly in lib/insecurity.ts (and the compiled copy under build). The private key is used to sign tokens.

**Why it matters.** Anyone with the private key can sign tokens that the application will trust, which means they can impersonate any user. In Juice Shop this is intentional and supports a token forging challenge.

**How to fix it.**

- Move the key out of source code and load it from an environment variable, a file outside the repository, or a secret manager.
- Rotate the key, because a key that has been committed must be treated as compromised.
- Add the key file pattern to .gitignore and add a secret scanner to the pre-commit hook so it cannot happen again.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#200](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/200) | `juice-shop/build/lib/insecurity.js` | 46 |
| [#203](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/203) | `juice-shop/lib/insecurity.ts` | 23 |
| [#103](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/103) | `lib/insecurity.ts` | 23 |

### JSON Web Token committed in test files

- Rule: `jwt-token`
- Severity: Medium
- Findings: 4

**What it is.** Full JWT strings are written into frontend test files (the .spec.ts files). These are test fixtures used to simulate a logged in user.

**Why it matters.** A real token in source can leak a real session if it is valid. Even a fake token trains people to paste real ones. The risk here is lower because these are test fixtures, not live secrets.

**How to fix it.**

- Replace the token with an obviously fake placeholder, or build a token at test time from a throwaway key.
- Never paste a real production token into a test file.
- Keep secret scanning on so a real token is caught before it is merged.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#201](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/201) | `juice-shop/frontend/src/app/app.guard.spec.ts` | 46 |
| [#202](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/202) | `juice-shop/frontend/src/app/last-login-ip/last-login-ip.component.spec.ts` | 72 |
| [#101](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/101) | `frontend/src/app/app.guard.spec.ts` | 46 |
| [#102](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/102) | `frontend/src/app/last-login-ip/last-login-ip.component.spec.ts` | 72 |
