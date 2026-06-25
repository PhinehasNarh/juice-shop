# Semgrep findings (pattern based)

Semgrep reported 28 findings. They are grouped below by rule. Several of these overlap with CodeQL findings (for example Sequelize injection and open redirect), which is normal and expected when two tools look at the same code.

### Sequelize injection (Semgrep)

- Rule id: `javascript.sequelize.security.audit.sequelize-injection-express.express-sequelize-injection`
- Severity: High
- Findings in this group: 6

**What it is.** A Sequelize statement is built from user input. This is the same underlying problem as SQL injection.

**How the scan found it.** Semgrep matched a Sequelize query that contains a value tied to request input without parameter binding.

**Why it matters.** An attacker can read or change database data, or bypass authentication.

**How to fix it.**

- Use replacements or bind parameters with sequelize.query, or use the model methods with a where object.
- Do not build query strings by concatenating request values.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#109](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/109) | `data/static/codefixes/dbSchemaChallenge_1.ts` | 5 |
| [#110](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/110) | `data/static/codefixes/dbSchemaChallenge_3.ts` | 11 |
| [#111](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/111) | `data/static/codefixes/unionSqlInjectionChallenge_1.ts` | 6 |
| [#112](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/112) | `data/static/codefixes/unionSqlInjectionChallenge_3.ts` | 10 |
| [#119](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/119) | `routes/login.ts` | 34 |
| [#123](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/123) | `routes/search.ts` | 23 |

### GitHub Actions shell injection

- Rule id: `yaml.github-actions.security.run-shell-injection.run-shell-injection`
- Severity: High
- Findings in this group: 5

**What it is.** A workflow run step uses ${{ github.* }} data directly inside a shell command. That context can contain attacker controlled text, for example a branch name or issue title.

**How the scan found it.** Semgrep matched a run: block that interpolates github context values straight into the shell.

**Why it matters.** An attacker can craft that text to inject shell commands into the runner, which can steal secrets and tamper with the build.

**How to fix it.**

- Pass the value through an env variable and reference it with quotes, for example set `env: TITLE: ${{ github.event.issue.title }}` then use "$TITLE" in the script.
- Never place ${{ ... }} github data directly inside a run command.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#104](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/104) | `.github/workflows/update-challenges-ebook.yml` | 22 |
| [#105](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/105) | `.github/workflows/update-challenges-www-legacy.yml` | 27 |
| [#106](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/106) | `.github/workflows/update-challenges-www-legacy.yml` | 36 |
| [#107](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/107) | `.github/workflows/update-challenges-www.yml` | 27 |
| [#108](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/108) | `.github/workflows/update-challenges-www.yml` | 36 |

### Web request data flowing into eval

- Rule id: `javascript.lang.security.audit.code-string-concat.code-string-concat`
- Severity: High
- Findings in this group: 1

**What it is.** Data from an Express request reaches eval. If the data is user controlled this runs as code.

**How the scan found it.** Semgrep matched request data flowing into eval.

**Why it matters.** Remote code execution in the server process.

**How to fix it.**

- Remove eval. Replace it with explicit logic or a safe parser.
- Never evaluate strings that contain request data.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#124](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/124) | `routes/userProfile.ts` | 61 |

### Directory listing enabled

- Rule id: `javascript.express.security.audit.express-check-directory-listing.express-check-directory-listing`
- Severity: Medium
- Findings in this group: 4

**What it is.** Static file serving is configured in a way that lets clients browse directory contents.

**How the scan found it.** Semgrep matched an Express static configuration that allows directory indexing.

**Why it matters.** Browsing the directory can reveal sensitive files and paths that were not meant to be public.

**How to fix it.**

- Disable directory listing. With serve-index do not enable it, and with express.static keep the default which does not list directories.
- Serve only the specific files you intend to expose.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#127](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/127) | `server.ts` | 269 |
| [#128](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/128) | `server.ts` | 273 |
| [#129](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/129) | `server.ts` | 277 |
| [#130](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/130) | `server.ts` | 281 |

### res.sendFile with user input

- Rule id: `javascript.express.security.audit.express-res-sendfile.express-res-sendfile`
- Severity: Medium
- Findings in this group: 4

**What it is.** A value from the request is passed to res.sendFile, which can be tricked into returning files outside the intended folder.

**How the scan found it.** Semgrep matched res.sendFile receiving user input.

**Why it matters.** Path traversal that lets an attacker read arbitrary files on the server.

**How to fix it.**

- Reduce the value to a base name and pass the root option, for example `res.sendFile(path.basename(name), { root: safeDir })`.
- Validate the requested file against an allow list before serving.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#116](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/116) | `routes/fileServer.ts` | 33 |
| [#117](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/117) | `routes/keyServer.ts` | 14 |
| [#118](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/118) | `routes/logfileServer.ts` | 14 |
| [#120](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/120) | `routes/quarantineServer.ts` | 14 |

### Unquoted template variable in an HTML attribute

- Rule id: `generic.html-templates.security.unquoted-attribute-var.unquoted-attribute-var`
- Severity: Medium
- Findings in this group: 3

**What it is.** A template puts a variable into an HTML attribute without quotes. An attacker controlled value can break out and add its own attributes or event handlers.

**How the scan found it.** Semgrep matched a template attribute such as href={{ value }} with no surrounding quotes.

**Why it matters.** Cross site scripting through injected attributes or JavaScript event handlers.

**How to fix it.**

- Quote the attribute, for example `href="{{ value }}"`.
- Make sure the template engine escapes the value for the HTML attribute context.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#113](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/113) | `frontend/src/app/navbar/navbar.component.html` | 17 |
| [#114](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/114) | `frontend/src/app/purchase-basket/purchase-basket.component.html` | 15 |
| [#131](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/131) | `views/dataErasureForm.hbs` | 38 |

### Unknown value used with a script tag

- Rule id: `javascript.lang.security.audit.unknown-value-with-script-tag.unknown-value-with-script-tag`
- Severity: Medium
- Findings in this group: 2

**What it is.** A value whose origin is not clear is written inside a script tag. If it can be influenced by a user, it runs as script.

**How the scan found it.** Semgrep matched a value placed inside a <script> block where the source could not be proven safe.

**Why it matters.** Cross site scripting, which can run code in the victim browser and steal sessions.

**How to fix it.**

- Do not place dynamic values inside script tags. Pass data through a JSON endpoint or a data attribute and read it with safe DOM APIs.
- If unavoidable, JSON encode and HTML escape the value for the script context.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#125](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/125) | `routes/videoHandler.ts` | 57 |
| [#126](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/126) | `routes/videoHandler.ts` | 71 |

### Open redirect (Semgrep)

- Rule id: `javascript.express.security.audit.express-open-redirect.express-open-redirect`
- Severity: Medium
- Findings in this group: 1

**What it is.** The app redirects to a URL taken from user input without validation.

**How the scan found it.** Semgrep matched a redirect whose target comes from the request.

**Why it matters.** Phishing through a trusted link that bounces to an attacker site.

**How to fix it.**

- Validate the target against an allow list of paths or hosts before redirecting.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#122](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/122) | `routes/redirect.ts` | 19 |

### Possible user input in a redirect (Semgrep)

- Rule id: `javascript.express.security.audit.possible-user-input-redirect.unknown-value-in-redirect`
- Severity: Medium
- Findings in this group: 1

**What it is.** A value that looks like it comes from user input is used as a redirect target.

**How the scan found it.** Semgrep matched a redirect target whose source could not be proven safe.

**Why it matters.** Open redirect that supports phishing.

**How to fix it.**

- Confirm the value is server controlled, or validate it against an allow list before redirecting.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#121](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/121) | `routes/redirect.ts` | 19 |

### Hardcoded JWT secret

- Rule id: `javascript.jsonwebtoken.security.jwt-hardcode.hardcoded-jwt-secret`
- Severity: Medium
- Findings in this group: 1

**What it is.** The secret or key used to sign or verify JSON Web Tokens is written directly in the source code.

**How the scan found it.** Semgrep matched a hardcoded credential passed to the jsonwebtoken library.

**Why it matters.** Anyone who reads the source, including from the public repository, can forge valid tokens and impersonate any user.

**How to fix it.**

- Load the secret from an environment variable or a secret manager, for example `process.env.JWT_SECRET`.
- Rotate the secret once it has been exposed, and keep secrets out of version control.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#115](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/115) | `lib/insecurity.ts` | 56 |
