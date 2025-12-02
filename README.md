# Security Scanning

## Overview
This repository uses three types of security scanning:

### SAST (Static Analysis with CodeQL)
- Scans source code for vulnerabilities
- Runs on push to main branch (and pull requests to main)
- Checks for:
  - Insecure Python/Flask coding patterns
  - Unsafe input handling and potential injection sinks
  - Hardcoded secrets or sensitive data
  - Misconfigurations (e.g. Flask debug mode enabled in production)
- Artifacts: See **GitHub Security → Code scanning alerts**
- Workflow file: .github/workflows/1-sast-only.yml

### SCA (Dependency Check)
- Scans Python packages for known CVEs
- Runs on push to main branch (and in the full security pipeline)
- Checks for:
  - Python dependencies with known CVEs
  - High/Critical vulnerabilities
  - Outdated or retired vulnerable libraries
- Artifacts: `sca-reports` (Dependency-Check HTML/JSON/CSV/XML reports)
- Workflow file: .github/workflows/2-sca-only.yml

### DAST (Live Application with ZAP)
- Tests running application for vulnerabilities
- Runs on push to main branch (after the app is started in CI)
- Checks for:
  - Missing or weak HTTP security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Permissions-Policy)
  - HTTP/HTTPS configuration issues (e.g. HTTP-only site)
  - Common web vulnerabilities on live endpoints (e.g. injection, XSS, information leakage)
- Artifacts: `zap-fullscan-reports/report_md.md`, `zap-baseline-reports/report_html.html`
- Workflow file: .github/workflows/3-dast-only.yml

## Understanding Results
### SAST Result (CodeQL)
- **Alert:** Flask app is run in debug mode  
- **Tool/Rule:** CodeQL – `py/flask-debug`  
- **Severity:** High  
- **Location:** `app.py` line 49 (`app.run(host="0.0.0.0", port=5000, debug=True)`)

**What this means:**  
Running a Flask app with `debug=True` enables the interactive Werkzeug debugger.  
If this is exposed in production, an attacker could use the debugger to execute arbitrary code on the server.

**Recommended fix:**  
Disable debug mode in production, for example:

```python
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=False)


## Reporting Vulnerabilities
Please email S10262458@connect.np.edu.sg