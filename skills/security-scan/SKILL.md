---
name: security-scan
description: >
  Security specialist focused on automated vulnerability scanning and SAST.
  Uses trivy for container and IaC scanning, and semgrep for static analysis.
  Follows Korvenix standards for security in CI/CD pipelines.
license: MIT
metadata:
  author: korvenix
  version: "1.0.0"
  domain: language
  triggers: security scan, trivy, semgrep, SAST, vulnerability, CVE, container scan, IaC scan, compliance
  role: specialist
  scope: implementation
  output-format: text
  related-skills: release-engineer, staff-engineer, paperclip
---

# Security Scan Skill

Expertise in automated security verification of container images, source code, and infrastructure-as-code.
Ensures critical vulnerabilities and insecure patterns are caught before deployment.

## Tech Stack (Non-Negotiable)

| Layer | Technology |
|-------|-----------|
| Container Scan | trivy |
| SAST | semgrep |
| IaC Scan | trivy (config scanning) |
| CI/CD | Cloud Build / GitHub Actions |
| Reporting | Cloud Logging / GitHub Security Alerts |

## When to Use This Skill

- Auditing a container image for CVEs before release
- Running static analysis on source code (Python, TypeScript, Go)
- Scanning Terraform files for insecure configurations (e.g., public buckets)
- Integrating security gates into CI/CD pipelines
- Investigating security alerts from automated scanners

## Core Workflow

1. **Source Analysis** — Run `semgrep` on source code to find insecure patterns (e.g., hardcoded secrets, SQL injection).
2. **IaC Analysis** — Run `trivy config` on Terraform modules to verify compliance with GCP security benchmarks.
3. **Container Scan** — Run `trivy image` on built container artifacts.
4. **Issue Triage** — Analyze findings, differentiate between true positives and false positives.
5. **Remediation** — Update dependencies, patch code, or adjust Terraform configs to resolve critical/high issues.
6. **Verification** — Re-run scans to confirm the fix.

## Scanning Patterns

### trivy Container Scan (cloudbuild.yaml)
```yaml
steps:
  - name: 'aquasec/trivy'
    args:
      - 'image'
      - '--severity'
      - 'HIGH,CRITICAL'
      - '--exit-code'
      - '1' # Fail the build on high/critical vulnerabilities
      - '${_REGION}-docker.pkg.dev/${PROJECT_ID}/${_REPOSITORY}/${_IMAGE}:${SHORT_SHA}'
```

### semgrep Static Analysis
```bash
# Run semgrep with default rules for Python/JS
semgrep --config auto --error .
```

### trivy Terraform Scan
```bash
# Scan terraform directory for misconfigurations
trivy config ./terraform
```

## Constraints

### MUST DO
- Fail CI/CD pipelines on any HIGH or CRITICAL severity vulnerability.
- Maintain a 'clean' baseline for all services.
- Update `trivy` and `semgrep` rules regularly.
- Use `semgrep --config auto` to leverage community-best-practice rulesets.
- Scan both application code and infrastructure (Terraform).

### MUST NOT DO
- Ignore HIGH/CRITICAL findings without a documented exception.
- Disable security gates in production pipelines.
- Store scan results in public locations.
- Hardcode security exceptions in source control (use ignore files with justifications).

## Reference Guide

| Topic | Load When |
|-------|-----------|
| trivy Advanced | Custom policies (Rego), air-gapped scans |
| semgrep Custom Rules | Writing domain-specific SAST rules |
| GCP Security Benchmarks | CIS Benchmarks for GCP, Google security blueprints |
| Dependency Management | Renovate/Dependabot integration |
