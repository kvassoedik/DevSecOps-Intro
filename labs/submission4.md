# Lab 4 — SBOM Generation & Software Composition Analysis

### Overview

In this lab, Software Bills of Materials (SBOMs) were generated for the
OWASP Juice Shop container image (`bkimminich/juice-shop:v19.0.0`).
Software Composition Analysis (SCA) was then performed using different
toolchains to evaluate dependency and vulnerability coverage.

Tools used:

- Syft (SBOM generation)
- Grype (vulnerability scanning)
- Trivy (SBOM + vulnerability + license scanning)

## Task 1 — SBOM Generation

### Docker Commands Used

Syft SBOM generation

```bash
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$(pwd)":/tmp anchore/syft:latest \
  bkimminich/juice-shop:v19.0.0 \
  -o syft-json=/tmp/labs/lab4/syft/syft.json
```

Trivy SBOM generation

```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$(pwd)":/tmp aquasec/trivy:latest image \
  --format json \
  --output /tmp/labs/lab4/trivy/trivy.json \
  --list-all-pkgs bkimminich/juice-shop:v19.0.0
```

### Quantitative Metrics
- Packages detected by Syft: 1139
- Packages detected by Trivy: 1135

### Qualitative Analysis

Syft provides very granular SBOM data, which is useful for compliance and deep supply-chain auditing.

Trivy integrates SBOM generation directly with security scanning, making it faster and easier for CI/CD pipelines.

## Task 2 — Software Composition Analysis (SCA)

### Docker Commands Used
Grype scan (using Syft SBOM)
```
docker run --rm -v "$(pwd)":/tmp anchore/grype:latest \
  sbom:/tmp/labs/lab4/syft/syft.json \
  -o table > labs/lab4/syft/grype.txt
```

Trivy vulnerability scan

```
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$(pwd)":/tmp aquasec/trivy:latest image \
  --format table \
  --output /tmp/labs/lab4/trivy/trivy-vuln.txt \
  bkimminich/juice-shop:v19.0.0
```

### Quantitative Vulnerability Metrics

- Critical vulnerabilities (Grype): 11
- Critical vulnerabilities (Trivy): 13

### Critical Vulnerability Analysis

Most vulnerabilities were related to:

- Outdated OS libraries
- Node.js dependencies
- Base image components

These findings indicate that the base container image is a primary
risk factor and should be regularly updated.

## Task 3 — Toolchain Comparison

### Accuracy & Coverage
- Syft detected more granular dependency metadata.
- Trivy detected vulnerabilities directly without requiring SBOM handoff.
- Overlapping vulnerabilities were observed between Grype and Trivy.

This shows both tools use different vulnerability databases and detection logic.

### Strengths & Weaknesses
#### Syft + Grype (Modular Approach)
Strengths:
- Detailed SBOM
- Better for compliance reporting
- Flexible integration

Weaknesses:
- Requires multiple tools
- Slightly more complex workflow

#### Trivy (All-in-One Approach)
Strengths:
- Simple to run
- Fast scanning
- Integrated vulnerability + license + secret scanning

Weaknesses:
- Less granular SBOM detail
- Less customizable pipeline separation

## Security Recommendations

Based on the findings, the following actions are recommended:

- Regularly update the base container image to reduce inherited OS vulnerabilities.
- Integrate Trivy scanning into CI/CD pipelines to detect issues early.
- Generate SBOMs (using Syft) for each release to maintain supply-chain visibility.
- Periodically review dependencies to remove outdated or unused libraries.
- Treat SBOMs as living artifacts and regenerate them after major dependency updates.

## Issues Encountered

- Initial Docker image downloads took additional time during setup.
- Output formats differed between tools, requiring manual comparison.
- Some vulnerabilities appeared in one scanner but not the other due to different vulnerability databases.

## Conclusion
This lab demonstrated how SBOM generation and SCA improve supply-chain
visibility and vulnerability management.

Trivy is well-suited for CI/CD automation.
Syft + Grype provide deeper inspection and compliance capabilities.

A combined approach offers the strongest security posture.