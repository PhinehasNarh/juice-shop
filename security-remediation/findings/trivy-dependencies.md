# Trivy findings: vulnerable dependencies

Trivy reported 72 dependency vulnerabilities across 23 packages. These are about the versions of third party packages installed under node_modules, not about code written in this repository. The fix for almost every one is the same idea: move the package to a version at or above the fixed version. Many of these are transitive, meaning they are pulled in by another package, so the cleanest fix is an overrides block in package.json. See [../02-remediation-summary.md](../02-remediation-summary.md) for a ready to paste block.

### jsonwebtoken

- Recommended version: **9.0.2 or newer**
- Findings for this package: 10
- Note: Token signing and verification library. Old 0.4.0 has signature bypass issues. Note that Juice Shop pins an old jsonwebtoken on purpose for a forging challenge.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#147](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/147) | CVE-2015-9235 | Critical | 4.2.2 |
| [#142](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/142) | CVE-2015-9235 | Critical | 4.2.2 |
| [#149](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/149) | NSWG-ECO-17 | High | >=4.2.2 |
| [#148](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/148) | CVE-2022-23539 | High | 9.0.0 |
| [#144](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/144) | NSWG-ECO-17 | High | >=4.2.2 |
| [#143](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/143) | CVE-2022-23539 | High | 9.0.0 |
| [#151](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/151) | CVE-2022-23541 | Medium | 9.0.0 |
| [#150](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/150) | CVE-2022-23540 | Medium | 9.0.0 |
| [#146](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/146) | CVE-2022-23541 | Medium | 9.0.0 |
| [#145](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/145) | CVE-2022-23540 | Medium | 9.0.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### lodash

- Recommended version: **4.17.21 or newer**
- Findings for this package: 5
- Note: Utility library pulled in by sanitize-html. Prototype pollution and command injection via template.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#154](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/154) | CVE-2019-10744 | Critical | 4.17.12 |
| [#156](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/156) | CVE-2021-23337 | High | 4.17.21 |
| [#155](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/155) | CVE-2018-16487 | High | >=4.17.11 |
| [#157](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/157) | CVE-2026-2950 | Medium | 4.18.0 |
| [#158](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/158) | CVE-2018-3721 | Low | >=4.17.5 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### crypto-js

- Recommended version: **4.2.0 or newer**
- Findings for this package: 1
- Note: Crypto helpers. PBKDF2 in 3.x is far weaker than expected. Juice Shop relies on this for a hashing challenge.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#136](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/136) | CVE-2023-46233 | Critical | 4.2.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### tar

- Recommended version: **7.5.16 or newer**
- Findings for this package: 15
- Note: Used transitively by build tooling (sqlite3, node-pre-gyp). Path traversal and overwrite issues.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#196](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/196) | CVE-2026-31802 | High | 7.5.11 |
| [#195](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/195) | CVE-2026-29786 | High | 7.5.10 |
| [#194](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/194) | CVE-2026-26960 | High | 7.5.8 |
| [#193](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/193) | CVE-2026-24842 | High | 7.5.7 |
| [#192](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/192) | CVE-2026-23950 | High | 7.5.4 |
| [#191](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/191) | CVE-2026-23745 | High | 7.5.3 |
| [#189](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/189) | CVE-2026-31802 | High | 7.5.11 |
| [#188](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/188) | CVE-2026-29786 | High | 7.5.10 |
| [#187](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/187) | CVE-2026-26960 | High | 7.5.8 |
| [#186](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/186) | CVE-2026-24842 | High | 7.5.7 |
| [#185](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/185) | CVE-2026-23950 | High | 7.5.4 |
| [#184](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/184) | CVE-2026-23745 | High | 7.5.3 |
| [#207](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/207) | CVE-2026-53655 | Medium | 7.5.16 |
| [#206](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/206) | CVE-2026-53655 | Medium | 7.5.16 |
| [#190](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/190) | CVE-2024-28863 | Medium | 6.2.1 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### multer

- Recommended version: **2.2.0 or newer**
- Findings for this package: 8
- Note: File upload middleware. Several denial of service issues fixed in 2.x.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#205](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/205) | CVE-2026-5079 | High | 2.2.0, 3.0.0-alpha.2 |
| [#172](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/172) | CVE-2026-3520 | High | 2.1.1 |
| [#171](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/171) | CVE-2026-3304 | High | 2.1.0 |
| [#170](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/170) | CVE-2026-2359 | High | 2.1.0 |
| [#169](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/169) | CVE-2025-7338 | High | 2.0.2 |
| [#168](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/168) | CVE-2025-48997 | High | 2.0.1 |
| [#167](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/167) | CVE-2025-47944 | High | 2.0.0 |
| [#166](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/166) | CVE-2025-47935 | High | 2.0.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### sanitize-html

- Recommended version: **2.12.1 or newer**
- Findings for this package: 8
- Note: HTML sanitizer. Old 1.4.2 has multiple cross site scripting bypasses. Juice Shop keeps an old version for an XSS challenge.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#173](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/173) | CVE-2022-25887 | High | 2.7.1 |
| [#180](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/180) | NSWG-ECO-154 | Medium | >=1.11.4 |
| [#179](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/179) | CVE-2024-21501 | Medium | 2.12.1 |
| [#178](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/178) | CVE-2021-26540 | Medium | 2.3.2 |
| [#177](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/177) | CVE-2021-26539 | Medium | 2.3.1 |
| [#176](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/176) | CVE-2019-25225 | Medium | 2.0.0-beta |
| [#175](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/175) | CVE-2017-16016 | Medium | 1.11.4 |
| [#174](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/174) | CVE-2016-1000237 | Medium | >=1.4.3 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### moment

- Recommended version: **2.30.1 or newer**
- Findings for this package: 3
- Note: Date library pulled in by express-jwt. ReDoS and path traversal in old versions.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#164](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/164) | CVE-2022-24785 | High | 2.29.2 |
| [#163](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/163) | CVE-2017-18214 | High | 2.19.3 |
| [#165](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/165) | CVE-2016-4055 | Medium | >=2.11.2 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### minimatch

- Recommended version: **3.1.4 or newer**
- Findings for this package: 3
- Note: Glob matcher pulled in by replace. ReDoS via catastrophic backtracking.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#162](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/162) | CVE-2026-27904 | High | 10.2.3, 9.0.7, 8.0.6, 7.4.8, 6.2.2, 5.1.8, 4.2.5, 3.1.4 |
| [#161](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/161) | CVE-2026-27903 | High | 10.2.3, 9.0.7, 8.0.6, 7.4.8, 6.2.2, 5.1.8, 4.2.5, 3.1.3 |
| [#160](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/160) | CVE-2026-26996 | High | 10.2.1, 9.0.6, 8.0.5, 7.4.7, 6.2.1, 5.1.7, 4.2.4, 3.1.3 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### ws

- Recommended version: **8.21.0 or newer**
- Findings for this package: 2
- Note: WebSocket library pulled in by socket.io. Denial of service from crafted frames and headers.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#208](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/208) | CVE-2026-48779 | High | 5.2.5, 6.2.4, 7.5.11, 8.21.0 |
| [#198](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/198) | CVE-2024-37890 | High | 5.2.4, 6.2.3, 7.5.10, 8.17.1 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### socket.io-parser

- Recommended version: **4.2.6 or newer**
- Findings for this package: 2
- Note: Encoder and decoder for socket.io. Denial of service and crash issues.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#183](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/183) | CVE-2023-32695 | High | 4.2.3, 3.4.3, 3.3.4 |
| [#182](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/182) | CVE-2026-33151 | High | 3.3.5, 3.4.4, 4.2.6 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### jws

- Recommended version: **4.0.0 or newer**
- Findings for this package: 2
- Note: JSON Web Signature library. Improper signature verification.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#153](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/153) | CVE-2025-65945 | High | 3.2.3, 4.0.1 |
| [#152](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/152) | CVE-2016-1000223 | High | >=3.0.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### base64url

- Recommended version: **3.0.0 or newer**
- Findings for this package: 2
- Note: Base64 URL helper. Out of bounds read and forgeable tokens in 0.0.6.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#133](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/133) | NSWG-ECO-428 | High | >=3.0.0 |
| [#134](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/134) | GHSA-rvg8-pwq2-xj7q | Medium | 3.0.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### uuid

- Recommended version: **11.1.1 or newer**
- Findings for this package: 1
- Note: Identifier library. Out of bounds write fixed in newer majors.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#197](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/197) | CVE-2026-41907 | High | 11.1.1, 12.0.1, 13.0.1 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### socket.io

- Recommended version: **4.6.2 or newer**
- Findings for this package: 1
- Note: Realtime engine. Unhandled error and resource issues.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#181](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/181) | CVE-2024-38355 | High | 2.5.1, 4.6.2 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### http-cache-semantics

- Recommended version: **4.1.1 or newer**
- Findings for this package: 1
- Note: HTTP caching helper. ReDoS issue.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#141](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/141) | CVE-2022-25881 | High | 4.1.1 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### express-jwt

- Recommended version: **8.4.1 or newer**
- Findings for this package: 1
- Note: JWT middleware. Old 0.1.3 has an authorization bypass. The 6.x and later API differs, so this needs a code change as well as an upgrade.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#138](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/138) | CVE-2020-15084 | High | 6.0.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### js-yaml

- Recommended version: **4.2.0 or newer**
- Findings for this package: 1
- Note: YAML parser. Crafted documents can cause issues in 3.x.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#204](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/204) | CVE-2026-53550 | Medium | 4.2.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### got

- Recommended version: **11.8.5 or newer**
- Findings for this package: 1
- Note: HTTP client. Redirect to UNIX socket and SSRF related issue.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#140](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/140) | CVE-2022-33987 | Medium | 12.1.0, 11.8.5 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### file-type

- Recommended version: **21.3.1 or newer**
- Findings for this package: 1
- Note: File type detection. Infinite loop denial of service. Note newer majors are ESM only.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#139](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/139) | CVE-2026-31808 | Medium | 21.3.1 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### engine.io

- Recommended version: **6.6.4 or newer**
- Findings for this package: 1
- Note: Transport under socket.io. Uncaught exception denial of service.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#137](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/137) | CVE-2022-41940 | Medium | 3.6.1, 6.2.1 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### messageformat

- Recommended version: **3.0.0 or newer**
- Findings for this package: 1
- Note: Message formatting. Prototype pollution in old version.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#159](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/159) | CVE-2025-57349 | Low | 3.0.0-beta.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### cookie

- Recommended version: **0.7.0 or newer**
- Findings for this package: 1
- Note: Cookie parser. Out of bounds character acceptance.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#135](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/135) | CVE-2024-47764 | Low | 0.7.0 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.

### @tootallnate/once

- Recommended version: **2.0.1 or newer**
- Findings for this package: 1
- Note: Promise helper. Minor denial of service.

| Alert | Advisory | Severity | Fixed version |
|------:|----------|----------|---------------|
| [#132](https://github.com/PhinehasNarh/juice-shop/security/code-scanning/132) | CVE-2026-3449 | Low | 3.0.1, 2.0.1 |

How to fix: upgrade the package to the recommended version. If it is a transitive dependency, add it to the overrides block in package.json and reinstall, then rerun the scan to confirm the alerts clear.
