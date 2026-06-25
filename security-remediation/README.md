# Security findings and remediation

This folder documents every finding from the automated security scan that runs in this repository, explains in plain language how each one was found, and gives a concrete fix for each one. It was produced for the OWASP Juice Shop fork at `PhinehasNarh/juice-shop`.

Scan date used for this report: 2026-06-25.

## Important context

OWASP Juice Shop is a deliberately insecure web application. It exists to teach people how vulnerabilities work, so many of these findings are intentional. The goal of this folder is not to silently remove the training value. The goal is to record, for each finding, what it is, how the scanner found it, what the risk would be in a real application, and exactly how you would fix it in real code. Where a finding is intentional for a Juice Shop challenge, the writeup says so.

## What the scan found

The scan reported **207 open alerts** in the GitHub Security tab. They come from three tools:

| Tool | What it checks | Findings |
|------|----------------|---------:|
| CodeQL | Static analysis of the application source code (SAST) | 100 |
| Semgrep OSS | Pattern based static analysis (SAST) | 28 |
| Trivy | Vulnerable dependencies and hardcoded secrets (SCA and secrets) | 79 |
| **Total** | | **207** |

By severity:

| Severity | Count |
|----------|------:|
| Critical | 11 |
| High | 136 |
| Medium | 56 |
| Low | 4 |

## How this folder is organized

1. [01-scan-methodology.md](01-scan-methodology.md). How the scan runs, which workflow produces it, and how a finding gets into the Security tab.
2. [findings/codeql-sast.md](findings/codeql-sast.md). All 100 CodeQL findings, grouped by rule, each with a fix.
3. [findings/semgrep-sast.md](findings/semgrep-sast.md). All 28 Semgrep findings, grouped by rule, each with a fix.
4. [findings/trivy-dependencies.md](findings/trivy-dependencies.md). All 72 vulnerable dependency findings, grouped by package, with the version to upgrade to.
5. [findings/trivy-secrets.md](findings/trivy-secrets.md). All 7 hardcoded secret findings.
6. [02-remediation-summary.md](02-remediation-summary.md). The consolidated set of fixes, including the dependency override block you can drop into package.json.
7. [03-how-to-close-a-finding-in-the-ui.md](03-how-to-close-a-finding-in-the-ui.md). Step by step on how to close or dismiss a finding yourself from the GitHub web interface.
8. [04-alert-index.md](04-alert-index.md). A single table listing all 207 alerts with their number, rule, severity, file, and disposition.

## How the findings were closed

Each alert in the Security tab was reviewed and given a disposition. The detail is in [04-alert-index.md](04-alert-index.md). Two alerts were left open on purpose so you can practice closing them yourself using the guide in [03-how-to-close-a-finding-in-the-ui.md](03-how-to-close-a-finding-in-the-ui.md).
