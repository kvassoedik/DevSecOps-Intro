# Lab 5 — Security Analysis: SAST & DAST of OWASP Juice Shop

## Overview

In this lab, static and dynamic security testing were performed against
OWASP Juice Shop (`bkimminich/juice-shop:v19.0.0`).

The goal was to compare:
- **SAST** with Semgrep
- **DAST** with ZAP, Nuclei, Nikto, and SQLmap

This allowed both source-code level and runtime security issues to be analyzed.

---

## Task 1 — Static Application Security Testing with Semgrep

### Commands Used

```bash
mkdir -p labs/lab5/{semgrep,zap,nuclei,nikto,sqlmap,analysis}
git clone https://github.com/juice-shop/juice-shop.git --depth 1 --branch v19.0.0 labs/lab5/semgrep/juice-shop

docker run --rm -v "$(pwd)/labs/lab5/semgrep/juice-shop":/src \
  -v "$(pwd)/labs/lab5/semgrep":/output \
  semgrep/semgrep:latest \
  semgrep --config=p/security-audit --config=p/owasp-top-ten \
  --json --output=/output/semgrep-results.json /src

docker run --rm -v "$(pwd)/labs/lab5/semgrep/juice-shop":/src \
  -v "$(pwd)/labs/lab5/semgrep":/output \
  semgrep/semgrep:latest \
  semgrep --config=p/security-audit --config=p/owasp-top-ten \
  --text --output=/output/semgrep-report.txt /src
```

### SAST Tool Effectiveness

Semgrep identified **25 findings** in the Juice Shop source code.

Semgrep was effective for detecting:
- insecure coding patterns
- potential injection risks
- dangerous API usage
- weak validation logic

SAST is useful because it finds issues directly in source code before deployment.

### Critical Vulnerability Analysis

Example critical findings from Semgrep:

| Vulnerability Type | File Path | Line | Severity |
|---|---|---:|---|
| SQL Injection (user-controlled query in Sequelize) | `/src/routes/login.ts` | 34 | Blocking |
| Insecure user-controlled input handling | `/src/lib/insecurity.ts` | 56 | Blocking |
| Missing validation / unsafe sink | `/src/routes/fileServer.ts` | 33 | Blocking |
| Weak cryptographic usage / insecure function | `/src/routes/redirect.ts` | 19 | Blocking |
| Hardcoded or unsafe sensitive logic | `/src/routes/userProfile.ts` | 62 | Blocking |

## Task 2 — Dynamic Application Security Testing with Multiple Tools

### Commands Used

#### Start Juice Shop

```bash
docker run -d --name juice-shop-lab5 -p 3000:3000 bkimminich/juice-shop:v19.0.0
sleep 10
curl -s http://localhost:3000 | head -n 5
```

#### ZAP Unauthenticated Scan

```bash
docker run --rm --network host \
  -v "$(pwd)/labs/lab5/zap":/zap/wrk/:rw \
  zaproxy/zap-stable:latest \
  zap-baseline.py -t http://localhost:3000 \
  -r report-noauth.html -J zap-report-noauth.json
```

#### ZAP Authenticated Scan

```bash
curl -s -X POST http://localhost:3000/rest/user/login \
  -H 'Content-Type: application/json' \
  -d '{"email":"admin@juice-sh.op","password":"admin123"}'

docker run --rm --network host \
  -v "$(pwd)/labs/lab5":/zap/wrk/:rw \
  zaproxy/zap-stable:latest \
  zap.sh -cmd -port 8090 \
  -autorun /zap/wrk/scripts/zap-auth.yaml
```

#### Nuclei

```bash
docker run --rm --network host \
  -v "$(pwd)/labs/lab5/nuclei":/app \
  projectdiscovery/nuclei:latest \
  -u http://localhost:3000 \
  -jsonl -o /app/nuclei-results.json
```

#### Nikto

```bash
docker run --rm --network host \
  -v "$(pwd)/labs/lab5/nikto":/tmp \
  alpine/nikto \
  -h http://localhost:3000 -o /tmp/nikto-results.txt
```

#### SQLmap

```bash
docker run --rm \
  --network host \
  -v "$(pwd)/labs/lab5/sqlmap":/output \
  secsi/sqlmap \
  -u "http://localhost:3000/rest/products/search?q=*" \
  --dbms=sqlite --batch --level=3 --risk=2 \
  --technique=B --threads=5 --output-dir=/output

docker run --rm \
  --network host \
  -v "$(pwd)/labs/lab5/sqlmap":/output \
  secsi/sqlmap \
  -u "http://localhost:3000/rest/user/login" \
  --data '{"email":"*","password":"test"}' \
  --method POST \
  --headers='Content-Type: application/json' \
  --dbms=sqlite --batch --level=5 --risk=3 \
  --technique=BT --threads=5 --output-dir=/output \
  --dump
```

### Authenticated vs Unauthenticated Scanning

Authenticated scanning is more valuable because it discovers:
- user-specific endpoints
- authenticated features
- admin functionality
- larger attack surface

Examples of authenticated/admin endpoints discovered:
- `/rest/admin/...`
- user account endpoints
- basket/order/payment routes

Authenticated scanning matters because many real vulnerabilities are only visible
after login.

### Tool Comparison Matrix

| Tool | Findings | Severity Breakdown | Best Use Case |
|---|---:|---|---|
| ZAP | 11 | High: 2, Medium: 4, Low: 5 | Broad web app scanning, authenticated workflows |
| Nuclei | 18 | Informational / template matches | Fast known-template / known-CVE checks |
| Nikto | 82 | Mostly informational server issues | Web server issues and misconfigurations |
| SQLmap | 1 | Critical (SQL Injection confirmed) | Deep SQL injection testing |

### Tool-Specific Strengths

#### ZAP
Best for:
- authenticated scanning
- crawling/spidering
- broad web application testing

Example findings:
- missing or weak security headers
- discovered authenticated/admin endpoints

#### Nuclei
Best for:
- fast template-based scans
- known weakness and known CVE detection

Example findings:
- template matches for exposed behaviors
- fast confirmation of known issues

#### Nikto
Best for:
- web server checks
- HTTP configuration and header issues

Example findings:
- missing hardening-related headers
- server information disclosure

#### SQLmap
Best for:
- SQL injection verification
- proving exploitability
- extracting affected database data when allowed in lab context

Example findings:
- injectable search parameter
- injectable login parameter

## Task 3 — SAST / DAST Correlation and Security Assessment

### SAST vs DAST Comparison

- **SAST findings:** 25
- **Combined DAST findings:** 112

### Vulnerabilities Found Mainly by SAST

SAST is better at finding:
- insecure code patterns
- unsafe function usage
- missing validation in source code
- risky developer logic before runtime

Examples:
- insecure input handling
- unsafe code paths
- weak crypto / risky API usage

### Vulnerabilities Found Mainly by DAST

DAST is better at finding:
- missing security headers
- authentication/session problems
- runtime misconfigurations
- exploitable SQL injection
- exposed endpoints and attack surface

Examples:
- admin endpoints only visible after login
- runtime response/header weaknesses
- live SQL injection behavior confirmed by SQLmap

### Why Each Approach Finds Different Things

SAST analyzes source code and can detect vulnerabilities before the application runs.
DAST interacts with the running application and finds issues that only appear in
deployed runtime behavior.

Because of this, both approaches are complementary and should be used together.

## Security Recommendations

Based on the findings, the following improvements are recommended:

1. Use Semgrep in CI/CD to catch insecure code before merge.
2. Use authenticated ZAP scans regularly in staging environments.
3. Use Nuclei for fast template-based checks in pipelines.
4. Use Nikto for periodic web server hardening validation.
5. Use SQLmap only for targeted SQL injection testing in controlled environments.
6. Review and fix injection-related code paths first.
7. Harden authentication, session handling, and security headers.

## Issues Encountered

- Some scans required additional setup files (such as ZAP auth configuration).
- Nuclei initially updated templates before running the actual scan.
- Tool output formats differed, so manual comparison was needed.
- Some DAST tools take significantly longer than others.

## Conclusion

This lab showed that SAST and DAST provide different but complementary security coverage.

- **Semgrep** is useful early in development.
- **ZAP** is the strongest general DAST tool, especially with authentication.
- **Nuclei** is useful for fast checks.
- **Nikto** is useful for server-level findings.
- **SQLmap** is the strongest tool for confirming SQL injection.

The best DevSecOps approach is to combine SAST and multiple DAST methods.