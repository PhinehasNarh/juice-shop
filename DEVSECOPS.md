# DevSecOps Pipeline — OWASP Juice Shop

A fully automated security scanning pipeline built on GitHub Actions, targeting OWASP Juice Shop as the test application. Demonstrates SAST, dependency scanning, and DAST integrated into a CI/CD workflow.

## Pipeline Overview

```
Push / Pull Request
        │
        ├─── SAST        → Semgrep      → GitHub Security tab (SARIF)
        ├─── SCA/Deps    → Trivy        → GitHub Security tab (SARIF)
        ├─── DAST        → OWASP ZAP    → Artifact (HTML/JSON report)
        └─── Summary     → Job summary  → Pass/fail dashboard
```

## Tools

| Layer | Tool | Purpose |
|-------|------|---------|
| SAST | [Semgrep](https://semgrep.dev) | Static code analysis — detects injection, hardcoded secrets, insecure patterns |
| Dependency Scan | [Trivy](https://trivy.dev) | Scans `package.json`, `node_modules`, and Docker image for CVEs |
| DAST | [OWASP ZAP](https://zaproxy.org) | Baseline scan against the running app — finds XSS, missing headers, open redirects |
| CI/CD | GitHub Actions | Orchestrates all jobs on every push and pull request |

## Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── devsecops.yml      # Main pipeline definition
├── .zap/
│   └── rules.tsv              # ZAP rule overrides (suppress known false positives)
├── security/
│   ├── findings.md            # Documented vulnerabilities with CVE refs and remediation
│   └── remediation/
│       ├── 001-sqli-fix.md
│       ├── 003-jwt-secret-fix.md
│       └── 006-security-headers-fix.md
└── README.md
```

## Pipeline Jobs

### 1. SAST — Semgrep

Runs on every push. Scans source code against the following rulesets:

- `p/security-audit` — general security anti-patterns
- `p/javascript` — JS-specific vulnerabilities
- `p/nodejs` — Node.js/Express-specific issues
- `p/owasp-top-ten` — mapped to OWASP Top 10

Results appear in the **Security → Code scanning** tab as SARIF.

### 2. Dependency Scan — Trivy

Runs two scans per pipeline execution:

1. **Filesystem scan** — parses `package.json`, `package-lock.json`, and lockfiles for vulnerable packages
2. **Container image scan** — builds the Docker image and scans all layers for OS and library CVEs

Filters to `HIGH` and `CRITICAL` severity only.

### 3. DAST — OWASP ZAP Baseline Scan

1. Pulls and starts the Juice Shop Docker container
2. Waits for the app to be ready (health-checks port 3000)
3. Runs ZAP Baseline Scan — passive + active spider, no authentication
4. Uploads HTML, JSON, and Markdown reports as workflow artifacts

The `.zap/rules.tsv` file suppresses known false positives so the report stays signal-focused.

### 4. Security Summary

Aggregates the pass/fail result of all three jobs into the workflow summary view.

## Key Findings

See [`security/findings.md`](security/findings.md) for the full documented findings. Highlights:

| ID | Vulnerability | Tool | Severity |
|----|--------------|------|----------|
| 001 | SQL Injection — Login bypass | Semgrep (SAST) | Critical |
| 002 | Reflected XSS — Search endpoint | ZAP (DAST) | High |
| 003 | Hardcoded JWT Secret | Semgrep (SAST) | Critical |
| 004 | IDOR — Broken Access Control | Manual (DAST-guided) | High |
| 005 | Vulnerable Dependencies (CVE-2022-23529, etc.) | Trivy (SCA) | Critical/High |
| 006 | Missing Security Headers | ZAP (DAST) | Medium |

## Running Locally

```bash
# SAST
docker run --rm -v $(pwd):/src semgrep/semgrep semgrep \
  --config=p/security-audit --config=p/javascript /src

# Dependency scan
trivy fs --severity HIGH,CRITICAL .

# DAST (requires Docker)
docker run -d -p 3000:3000 bkimminich/juice-shop
docker run -t owasp/zap2docker-stable zap-baseline.py \
  -t http://host.docker.internal:3000 -r zap-report.html
```

## References

- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)
- [Semgrep Rules Registry](https://semgrep.dev/r)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [OWASP ZAP Baseline Scan](https://www.zaproxy.org/docs/docker/baseline-scan/)
- [OWASP Top 10 (2021)](https://owasp.org/Top10/)
