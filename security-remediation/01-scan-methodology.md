# How the scan works and how the findings were found

This page explains where these findings come from, so the rest of the folder makes sense.

## The pipeline that produces the findings

The scan is part of the DevSecOps workflow in this repository, at `.github/workflows/devsecops.yml`. It runs on pushes and on a schedule. The workflow runs three independent security tools and uploads each tool's results to GitHub code scanning. Once uploaded, every result becomes an alert in the repository Security tab under Code scanning.

The three tools play different roles.

### 1. CodeQL (application code analysis)

CodeQL builds a database that represents the application source code, then runs queries against it. Each query looks for a known weakness pattern. The powerful part is data flow tracking. CodeQL follows a value from where it enters the program, called a source, for example `req.query.id`, all the way to a sensitive operation, called a sink, for example a database query or a file read. If a value can travel from a source to a sink without being made safe along the way, CodeQL raises an alert.

That is why a CodeQL finding tells you both a location and a story. For example, the SQL injection alerts say a request value reached a database query with no parameter binding in between.

### 2. Semgrep OSS (pattern based analysis)

Semgrep matches code against rules that describe risky patterns. It is faster and more surface level than CodeQL. It is good at catching specific shapes of code, for example a Sequelize query built from input, a redirect that uses a request value, or a GitHub Actions step that drops untrusted context straight into a shell command.

### 3. Trivy (dependencies and secrets)

Trivy does two jobs here.

- Software composition analysis. It reads the installed packages under node_modules and compares each package and version against a database of known vulnerabilities. When the installed version is in the vulnerable range, it reports the advisory and the version that fixes it. These findings point at a `package.json` inside node_modules, not at your own code.
- Secret scanning. It looks for things that look like credentials committed in the repository, for example a private key block or a JSON Web Token. These findings point at the file that contains the secret.

## How to read a finding

Every finding in this folder has the same shape.

- What it is. A plain description of the weakness.
- How the scan found it. What the tool actually matched or traced.
- Why it matters. The real world risk if this were a production application.
- How to fix it. The concrete change you would make.
- A link to the exact alert and the file and line.

## Why so many findings, and why that is expected here

Juice Shop is built to be vulnerable, so the application code findings from CodeQL and Semgrep are mostly intentional teaching examples. The dependency findings from Trivy are a mix. Some are intentional, because a few packages are pinned to old versions for specific challenges. Many others are simply old transitive dependencies that have known advisories. The remediation summary explains which ones are safe to upgrade and which ones are deliberately kept old.
