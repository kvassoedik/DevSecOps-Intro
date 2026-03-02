# Lab 3 — Secure Git

## Overview

This lab focused on implementing secure Git practices to protect the software
development lifecycle. Two main controls were introduced:

1. SSH commit signing to verify authenticity and integrity of commits.
2. Automated secret scanning using pre-commit hooks with TruffleHog and Gitleaks.

These controls simulate real DevSecOps protections used to prevent supply-chain
attacks and accidental credential leaks.

---

## Task 1 — SSH Commit Signing

### Configuration

An Ed25519 SSH key was generated and added to GitHub as an SSH **Signing Key**.
Git was configured to sign commits automatically using SSH format:

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgSign true
```

A signed commit was created using:

```bash
git commit -S -m "test: signed commit"
```

### Verification
The commit shows the Verified badge on GitHub, proving that:
- The commit author is authenticated.
- The commit content has not been altered.
- The repository can trust contributions from verified identities.

### Why Commit Signing Matters in DevSecOps
Commit signing prevents impersonation attacks and protects against malicious code injection. In CI/CD pipelines, only signed commits can be trusted, ensuring traceability and integrity of software changes.

## Task 2 — Pre-commit Secret Scanning

### Hook Implementation
A local Git pre-commit hook was created in:
```bash
.git/hooks/pre-commit
```
The hook runs Dockerized security scanners:
- TruffleHog — detects high-entropy secrets and credentials
- Gitleaks — detects known secret patterns
These tools scan staged files before allowing a commit.

### Testing the Hook
A test secret was added to a file and staged.

The pre-commit hook detected the secret and blocked the commit:
```bash
9:08PM INF scanned ~287 bytes (287 bytes) in 45.4ms
9:08PM WRN leaks found: 1
---
✖ Secrets found in non-excluded file: secret.txt

[pre-commit] === SCAN SUMMARY ===
TruffleHog found secrets in non-lectures files: false
Gitleaks found secrets in non-lectures files: true
Gitleaks found secrets in lectures files: false

✖ COMMIT BLOCKED: Secrets detected in non-excluded files.
Fix or unstage the offending files and try again.
```
### Security Value
This automated control prevents developers from accidentally committing:
- API keys
- Tokens
- Passwords
- Private keys

Secret leakage is one of the most common causes of real-world breaches, and pre-commit scanning stops exposure before it reaches the repository.

## Conclusion
By combining commit signing and automated secret detection, this lab demonstrated how DevSecOps integrates security directly into developer workflows.

These protections:
- Ensure trusted code provenance
- Prevent sensitive data exposure
- Reduce supply-chain risk
- Shift security left into development

Secure Git practices are essential for maintaining integrity in collaborative and automated environments.

