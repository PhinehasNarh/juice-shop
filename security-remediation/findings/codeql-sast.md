# CodeQL findings (application code)

CodeQL reported 100 findings in the application source code. They are grouped below by rule, ordered roughly from most to least severe. Each group explains the weakness once and then lists every affected location with a link to the alert.

### Type confusion through parameter tampering

- Rule id: `js/type-confusion-through-parameter-tampering`
- Severity: Critical
- Findings in this group: 2

**What it is.** A request parameter is expected to be a string, but the framework can deliver it as an array (for example id[]=1&id[]=2). Code that assumes a string then behaves in surprising ways.

**How the scan found it.** CodeQL saw an HTTP parameter used as a string in a place where it could also arrive as an array.

**Why it matters.** The mismatch can bypass validation or trigger logic the developer did not expect, which can break authentication or filters.

**How to fix it.**

- Check the type before use, for example `if (typeof v !== 'string') return res.sendStatus(400)`.
- Normalize with `Array.isArray(v) ? v[0] : v`, or use a schema validator such as zod or joi.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#37](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/37) | `lib/insecurity.ts` | 138 |
| [#38](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/38) | `routes/search.ts` | 22 |

### Template object injection

- Rule id: `js/template-object-injection`
- Severity: Critical
- Findings in this group: 2

**What it is.** A user supplied value is placed into the object that drives a template. The user can control template fields they should not.

**How the scan found it.** CodeQL traced request input into the data object passed to a template engine.

**Why it matters.** Depending on the engine this can lead to information disclosure or template injection that runs code.

**How to fix it.**

- Build the template context yourself from known fields, do not spread user input into it.
- Validate the keys and values that go into the context against an allow list.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#9](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/9) | `routes/dataErasure.ts` | 107 |
| [#10](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/10) | `routes/dataErasure.ts` | 123 |

### Code injection

- Rule id: `js/code-injection`
- Severity: Critical
- Findings in this group: 2

**What it is.** User input reaches a dynamic code evaluation such as eval or the Function constructor. The input is run as code.

**How the scan found it.** CodeQL traced request data into eval or a similar execute sink.

**Why it matters.** Remote code execution. This is one of the most serious issues because the attacker can run arbitrary code in the server process.

**How to fix it.**

- Never pass user input to eval, Function, setTimeout with a string, or similar.
- Replace dynamic evaluation with a safe parser or an explicit allow list of operations.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#4](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/4) | `routes/showProductReviews.ts` | 36 |
| [#3](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/3) | `routes/trackOrder.ts` | 18 |

### Server side request forgery (SSRF)

- Rule id: `js/request-forgery`
- Severity: Critical
- Findings in this group: 1

**What it is.** The server makes an outbound HTTP request to a URL that the user controls.

**How the scan found it.** CodeQL traced a request value into the URL of an outbound fetch made by the server.

**Why it matters.** An attacker can make the server call internal services, cloud metadata endpoints, or other hosts behind the firewall, which can leak credentials or reach private systems.

**How to fix it.**

- Validate the URL and allow only an explicit list of hosts and schemes.
- Block requests to private and loopback IP ranges and to the cloud metadata address.
- Resolve the host and re-check it to avoid DNS rebinding.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#6](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/6) | `routes/profileImageUrlUpload.ts` | 24 |

### Missing rate limiting

- Rule id: `js/missing-rate-limiting`
- Severity: High
- Findings in this group: 35

**What it is.** A route handler reaches out to the file system or the database, but there is no limit on how many times a single client can call it. A caller can hit the endpoint as fast as they want.

**How the scan found it.** CodeQL traced each Express route in server.ts and saw that the handler performs a file system or database access while no rate limiting middleware sits in front of it.

**Why it matters.** An attacker can hammer the endpoint to exhaust CPU, disk, or database connections, which is a denial of service. Unlimited requests also make brute force and scraping cheap.

**How to fix it.**

- Add a shared rate limiter such as the express-rate-limit package and apply it to the sensitive routes.
- Example: `import rateLimit from 'express-rate-limit'` then `const limiter = rateLimit({ windowMs: 15*60*1000, max: 100 })` and mount it with `app.use('/api/', limiter)` or per route.
- Put stricter limits on login, password change, file read, and file upload routes.
- Behind a proxy, set `app.set('trust proxy', 1)` so the client IP is read correctly.

**Juice Shop note.** server.ts wires up most of the application routes in one place, so all 35 of these point back to the same file. One central limiter plus a few per route limiters clears the whole group.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#66](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/66) | `server.ts` | 270 |
| [#67](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/67) | `server.ts` | 271 |
| [#68](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/68) | `server.ts` | 278 |
| [#69](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/69) | `server.ts` | 283 |
| [#70](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/70) | `server.ts` | 309 |
| [#71](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/71) | `server.ts` | 310 |
| [#72](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/72) | `server.ts` | 311 |
| [#73](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/73) | `server.ts` | 430 |
| [#74](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/74) | `server.ts` | 462 |
| [#75](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/75) | `server.ts` | 595 |
| [#77](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/77) | `server.ts` | 599 |
| [#76](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/76) | `server.ts` | 599 |
| [#78](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/78) | `server.ts` | 616 |
| [#79](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/79) | `server.ts` | 620 |
| [#80](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/80) | `server.ts` | 621 |
| [#81](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/81) | `server.ts` | 622 |
| [#83](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/83) | `server.ts` | 623 |
| [#82](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/82) | `server.ts` | 623 |
| [#85](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/85) | `server.ts` | 624 |
| [#84](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/84) | `server.ts` | 624 |
| [#86](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/86) | `server.ts` | 628 |
| [#87](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/87) | `server.ts` | 631 |
| [#88](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/88) | `server.ts` | 632 |
| [#89](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/89) | `server.ts` | 633 |
| [#90](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/90) | `server.ts` | 634 |
| [#91](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/91) | `server.ts` | 650 |
| [#92](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/92) | `server.ts` | 651 |
| [#93](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/93) | `server.ts` | 652 |
| [#94](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/94) | `server.ts` | 661 |
| [#95](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/95) | `server.ts` | 662 |
| [#97](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/97) | `server.ts` | 665 |
| [#96](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/96) | `server.ts` | 665 |
| [#98](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/98) | `server.ts` | 666 |
| [#99](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/99) | `server.ts` | 670 |
| [#100](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/100) | `server.ts` | 672 |

### Path injection (uncontrolled data used in a path)

- Rule id: `js/path-injection`
- Severity: High
- Findings in this group: 11

**What it is.** A value that came from the request is used to build a file system path. By sending values like ../../etc/passwd, a caller can step outside the intended folder.

**How the scan found it.** CodeQL followed the tainted value from the request (params, query, body, or an uploaded name) into a call such as fs.readFile, fs.writeFile, or path.join, with no normalization in between.

**Why it matters.** Directory traversal. An attacker can read or write files outside the intended directory, which can leak secrets or overwrite application files.

**How to fix it.**

- Reduce the input to a bare file name with `path.basename(userValue)` before using it.
- Resolve the final path and confirm it still sits inside the allowed base folder: `const full = path.resolve(baseDir, name); if (!full.startsWith(path.resolve(baseDir)+path.sep)) throw new Error('invalid path')`.
- Where possible, map the user value to a known key in an allow list instead of using it directly as a path.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#25](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/25) | `routes/fileServer.ts` | 33 |
| [#27](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/27) | `routes/fileUpload.ts` | 33 |
| [#28](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/28) | `routes/fileUpload.ts` | 38 |
| [#26](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/26) | `routes/keyServer.ts` | 14 |
| [#29](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/29) | `routes/logfileServer.ts` | 14 |
| [#31](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/31) | `routes/profileImageUrlUpload.ts` | 29 |
| [#30](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/30) | `routes/quarantineServer.ts` | 14 |
| [#32](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/32) | `routes/vulnCodeFixes.ts` | 80 |
| [#33](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/33) | `routes/vulnCodeFixes.ts` | 81 |
| [#34](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/34) | `routes/vulnCodeSnippet.ts` | 89 |
| [#35](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/35) | `routes/vulnCodeSnippet.ts` | 90 |

### SQL injection

- Rule id: `js/sql-injection`
- Severity: High
- Findings in this group: 7

**What it is.** A database query is built by gluing user input straight into the SQL string. Crafted input changes the meaning of the query.

**How the scan found it.** CodeQL traced request data into a raw query (sequelize.query or a string passed to the database) without a parameter binding step.

**Why it matters.** An attacker can read, change, or delete any data in the database, bypass login, or in some setups run commands on the database host.

**How to fix it.**

- Use parameterized queries. With Sequelize use replacements or bind parameters: `sequelize.query('... WHERE id = :id', { replacements: { id }, type: QueryTypes.SELECT })`.
- Prefer the model layer (Model.findOne({ where: { id } })) which parameterizes for you.
- Never concatenate request values into SQL text. Validate and type cast numeric ids before use.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#43](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/43) | `routes/likeProductReviews.ts` | 25 |
| [#44](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/44) | `routes/likeProductReviews.ts` | 36 |
| [#45](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/45) | `routes/likeProductReviews.ts` | 43 |
| [#46](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/46) | `routes/likeProductReviews.ts` | 51 |
| [#47](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/47) | `routes/login.ts` | 34 |
| [#48](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/48) | `routes/search.ts` | 23 |
| [#49](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/49) | `routes/updateProductReviews.ts` | 18 |

### Remote property injection

- Rule id: `js/remote-property-injection`
- Severity: High
- Findings in this group: 5

**What it is.** A property name to read or write on an object is taken from user input. A caller can target unexpected properties, including special ones like __proto__.

**How the scan found it.** CodeQL traced a request value into a computed property access such as obj[userKey] = value.

**Why it matters.** An attacker can overwrite fields they should not touch, tamper with application state, or pollute the object prototype, which can lead to logic bypass or denial of service.

**How to fix it.**

- Check the key against an allow list of permitted property names before writing.
- Reject keys equal to __proto__, prototype, or constructor.
- Use a Map, or `Object.create(null)`, when you must store user supplied keys.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#56](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/56) | `lib/accuracy.ts` | 64 |
| [#57](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/57) | `lib/insecurity.ts` | 76 |
| [#58](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/58) | `lib/insecurity.ts` | 77 |
| [#59](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/59) | `routes/currentUser.ts` | 31 |
| [#60](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/60) | `routes/vulnCodeFixes.ts` | 40 |

### User controlled bypass of a security check

- Rule id: `js/user-controlled-bypass`
- Severity: High
- Findings in this group: 4

**What it is.** A decision that guards a sensitive action depends on a value the user can set. The user can flip the flag and skip the check.

**How the scan found it.** CodeQL found a conditional that protects a sensitive operation where the condition is influenced by request input.

**Why it matters.** Authorization or validation bypass. The user can reach actions or data that should be blocked.

**How to fix it.**

- Base security decisions on server side state, for example the authenticated session and roles, not on values sent by the client.
- Re-check permissions on the server for every sensitive action.
- Treat any client provided flag as a hint only, never as the source of truth.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#61](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/61) | `lib/insecurity.ts` | 57 |
| [#62](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/62) | `lib/insecurity.ts` | 190 |
| [#63](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/63) | `routes/fileServer.ts` | 27 |
| [#64](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/64) | `routes/verify.ts` | 112 |

### Incomplete URL substring check

- Rule id: `js/incomplete-url-substring-sanitization`
- Severity: High
- Findings in this group: 4

**What it is.** Code checks a URL by looking for a substring such as indexOf('example.com'). An attacker can place that text anywhere in a longer URL they control.

**How the scan found it.** CodeQL saw a host or URL check done with includes or indexOf rather than by parsing the URL and comparing the host.

**Why it matters.** The check can be fooled, which enables open redirects, server side request forgery, or trusting an attacker domain like example.com.attacker.tld.

**How to fix it.**

- Parse the URL with `new URL(value)` and compare `url.hostname` exactly against an allow list.
- Anchor any host comparison to the full hostname, not a substring.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#39](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/39) | `routes/updateUserProfile.ts` | 31 |
| [#40](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/40) | `routes/updateUserProfile.ts` | 32 |
| [#41](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/41) | `test/api/internet-resources.test.ts` | 65 |
| [#42](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/42) | `test/api/internet-resources.test.ts` | 81 |

### Incomplete string escaping or encoding

- Rule id: `js/incomplete-sanitization`
- Severity: High
- Findings in this group: 3

**What it is.** A replace call only changes the first match, so other dangerous characters survive. Sanitizing once is not enough when the input can contain several.

**How the scan found it.** CodeQL saw a `String.replace` with a non global pattern used to clean input, which replaces only the first occurrence.

**Why it matters.** Characters that should have been escaped slip through, which can lead to injection such as SQL injection or cross site scripting.

**How to fix it.**

- Use a global regular expression, for example `value.replace(/['";]/g, '')`, or better a proper encoder for the target context.
- Do not hand roll escaping. Use parameterized queries for SQL and a trusted HTML encoder for output.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#65](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/65) | `data/static/codefixes/unionSqlInjectionChallenge_1.ts` | 5 |
| [#11](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/11) | `lib/utils.ts` | 47 |
| [#12](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/12) | `server.ts` | 256 |

### Clear text logging of sensitive information

- Rule id: `js/clear-text-logging`
- Severity: High
- Findings in this group: 2

**What it is.** Sensitive data such as a password is written to the logs in plain text.

**How the scan found it.** CodeQL traced sensitive values into a logging call.

**Why it matters.** Anyone who can read the logs, including operators and anyone who breaches the log store, sees the secret.

**How to fix it.**

- Never log passwords, tokens, or keys. Remove or redact them before logging.
- Log an identifier or a hash if you need to correlate, not the secret itself.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#7](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/7) | `lib/antiCheat.ts` | 64 |
| [#8](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/8) | `lib/antiCheat.ts` | 93 |

### Missing regular expression anchor

- Rule id: `js/regex/missing-regexp-anchor`
- Severity: High
- Findings in this group: 1

**What it is.** A regular expression used to validate a URL or host is not anchored with ^ and $, so it matches anywhere in the string.

**How the scan found it.** CodeQL saw a URL being validated by a regex that has no start or end anchor.

**Why it matters.** An attacker can prepend or append text to slip past the check, which enables open redirect or host spoofing.

**How to fix it.**

- Anchor the pattern, for example `^https://allowed\.example\.com/`.
- For host checks, parse the URL and compare the hostname exactly instead of using a regex.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#54](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/54) | `test/cypress/e2e/redirect.spec.ts` | 19 |

### File system race condition (time of check to time of use)

- Rule id: `js/file-system-race`
- Severity: High
- Findings in this group: 1

**What it is.** The code checks a file (for example that it exists) and then acts on it in a separate step. The file can change between the two steps.

**How the scan found it.** CodeQL saw a check such as fs.existsSync followed by a later use of the same path.

**Why it matters.** An attacker who can change the file or swap a symlink in the gap can make the program act on the wrong file.

**How to fix it.**

- Do the operation directly and handle the error, instead of checking first and acting later.
- Open the file once and work with the returned handle rather than the path.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#51](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/51) | `lib/codingChallenges.ts` | 29 |

### Insecure randomness

- Rule id: `js/insecure-randomness`
- Severity: High
- Findings in this group: 1

**What it is.** Math.random is used to produce a value that needs to be unpredictable, such as a token. Math.random is not cryptographically secure.

**How the scan found it.** CodeQL saw Math.random feeding a security sensitive value.

**Why it matters.** An attacker can predict the value, which can let them guess tokens, reset codes, or identifiers.

**How to fix it.**

- Use the crypto module, for example `crypto.randomBytes(32).toString('hex')` or `crypto.randomUUID()`.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#50](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/50) | `lib/insecurity.ts` | 55 |

### Weak password hashing

- Rule id: `js/insufficient-password-hash`
- Severity: High
- Findings in this group: 1

**What it is.** Passwords are hashed with a fast algorithm such as MD5 or a plain SHA, which is far too quick and easy to crack.

**How the scan found it.** CodeQL traced a password value into a weak hash function.

**Why it matters.** If the database leaks, attackers crack the hashes quickly and recover the plain passwords.

**How to fix it.**

- Use a slow, salted password hash such as bcrypt, scrypt, or argon2, for example `await bcrypt.hash(password, 12)`.
- Verify with the matching compare function, never by hashing and comparing strings yourself.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#36](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/36) | `lib/insecurity.ts` | 43 |

### Zip Slip (arbitrary file write during archive extraction)

- Rule id: `js/zipslip`
- Severity: High
- Findings in this group: 1

**What it is.** An archive is extracted and an entry name that contains ../ is used to build the output path. The entry can escape the target folder.

**How the scan found it.** CodeQL traced an archive entry name into a file system write during extraction.

**Why it matters.** An attacker who supplies a crafted archive can write files anywhere the process can write, which can lead to code execution.

**How to fix it.**

- For every entry, resolve the target and confirm it stays inside the extraction directory before writing.
- Reject any entry whose resolved path does not start with the destination folder.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#24](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/24) | `routes/fileUpload.ts` | 41 |

### Missing CSRF protection

- Rule id: `js/missing-token-validation`
- Severity: High
- Findings in this group: 1

**What it is.** A state changing route is served behind cookie based sessions but has no cross site request forgery protection.

**How the scan found it.** CodeQL saw cookie middleware serving a handler that changes state with no CSRF token check.

**Why it matters.** A malicious page can make the victim's browser send authenticated requests, performing actions as the victim.

**How to fix it.**

- Add CSRF protection, for example the csurf middleware, and require a valid token on state changing requests.
- Set session cookies with SameSite=Lax or Strict and the Secure flag.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#23](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/23) | `server.ts` | 289 |

### Regular expression denial of service (ReDoS)

- Rule id: `js/polynomial-redos`
- Severity: High
- Findings in this group: 1

**What it is.** A regular expression runs on user input and has a pattern that can take a very long time on certain strings, for example many repeats of one character.

**How the scan found it.** CodeQL saw user input flow into a regular expression whose structure can backtrack heavily.

**Why it matters.** A small crafted input can pin the CPU and freeze the server, which is a denial of service.

**How to fix it.**

- Simplify the pattern to avoid nested or overlapping quantifiers.
- Cap the input length before matching, or use a linear time engine such as RE2.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#1](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/1) | `routes/profileImageUrlUpload.ts` | 20 |

### Stack trace exposure

- Rule id: `js/stack-trace-exposure`
- Severity: Medium
- Findings in this group: 7

**What it is.** When an error happens, the raw error or its stack trace is sent back in the HTTP response. The stack reveals file paths, library versions, and internal logic.

**How the scan found it.** CodeQL saw error or error.stack flowing from a catch block into the response body that is returned to the client.

**Why it matters.** Information disclosure. The detail helps an attacker map the code and find more weaknesses. It can also leak data inside error messages.

**How to fix it.**

- Return a short, generic message to the client, for example `res.status(500).json({ error: 'Internal server error' })`.
- Log the full error on the server only, through your logger.
- Add a central Express error handler so no route leaks the raw error.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#16](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/16) | `routes/checkKeys.ts` | 30 |
| [#17](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/17) | `routes/checkKeys.ts` | 39 |
| [#18](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/18) | `routes/createProductReviews.ts` | 32 |
| [#19](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/19) | `routes/likeProductReviews.ts` | 56 |
| [#20](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/20) | `routes/nftMint.ts` | 33 |
| [#21](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/21) | `routes/nftMint.ts` | 50 |
| [#22](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/22) | `routes/web3Wallet.ts` | 36 |

### Sensitive data read from a GET query parameter

- Rule id: `js/sensitive-get-query`
- Severity: Medium
- Findings in this group: 3

**What it is.** A secret such as a password or token is read from the URL query string of a GET request.

**How the scan found it.** CodeQL saw req.query used as sensitive data inside a GET handler.

**Why it matters.** Query strings are logged by servers, proxies, and browser history, so the secret leaks into many places.

**How to fix it.**

- Move secrets into the request body of a POST, or into a header, never the URL.
- Make sure access logs do not record sensitive query values.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#13](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/13) | `routes/changePassword.ts` | 14 |
| [#14](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/14) | `routes/changePassword.ts` | 15 |
| [#15](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/15) | `routes/changePassword.ts` | 17 |

### Network data written to a file

- Rule id: `js/http-to-file-access`
- Severity: Medium
- Findings in this group: 2

**What it is.** Data fetched from the network or sent by the user is written to disk without checking its content, size, or destination.

**How the scan found it.** CodeQL traced untrusted data into a file write operation.

**Why it matters.** An attacker can fill the disk, plant malicious files, or combine this with path injection to write to a chosen location.

**How to fix it.**

- Validate the content type and enforce a maximum size before writing.
- Write to a fixed directory with a generated file name, not a user supplied one.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#52](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/52) | `routes/fileUpload.ts` | 35 |
| [#53](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/53) | `routes/profileImageFileUpload.ts` | 43 |

### Log injection

- Rule id: `js/log-injection`
- Severity: Medium
- Findings in this group: 1

**What it is.** User input is written to the log without removing newlines, so an attacker can forge extra log lines.

**How the scan found it.** CodeQL traced request input into a log message with no encoding.

**Why it matters.** Forged or split log entries can hide an attack, mislead an investigation, or break log parsers.

**How to fix it.**

- Strip or encode carriage return and newline characters before logging user input, for example `value.replace(/[\r\n]/g, ' ')`.
- Prefer structured logging where the message and the data are separate fields.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#55](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/55) | `lib/challengeUtils.ts` | 96 |

### Open redirect

- Rule id: `js/server-side-unvalidated-url-redirection`
- Severity: Medium
- Findings in this group: 1

**What it is.** The server redirects the browser to a URL taken from user input without checking it.

**How the scan found it.** CodeQL traced a request value into a redirect location.

**Why it matters.** An attacker can send a link to your site that bounces the victim to a malicious site, which helps phishing.

**How to fix it.**

- Redirect only to an allow list of known paths or hosts.
- If you must allow arbitrary targets, show a warning interstitial first.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#5](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/5) | `routes/redirect.ts` | 19 |

### Prototype polluting assignment

- Rule id: `js/prototype-polluting-assignment`
- Severity: Medium
- Findings in this group: 1

**What it is.** An assignment uses a user controlled key, so a key like __proto__ can change Object.prototype and affect every object.

**How the scan found it.** CodeQL saw a write where the property name comes from user input and is not guarded against __proto__.

**Why it matters.** Prototype pollution can cause denial of service, logic bypass, and in some cases code execution.

**How to fix it.**

- Reject the keys __proto__, prototype, and constructor.
- Use a Map or `Object.create(null)` for user supplied keys, and freeze objects you do not expect to change.

Affected locations:

| Alert | File | Line |
|------:|------|-----:|
| [#2](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/2) | `lib/accuracy.ts` | 67 |
