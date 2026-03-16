# Lab 6 - Infrastructure-as-Code Security: Scanning & Policy Enforcement

## Overview

In this lab, Infrastructure-as-Code security scanning was performed on
intentionally vulnerable Terraform, Pulumi, and Ansible configurations.

Security scanners were used to detect misconfigurations and insecure
infrastructure definitions.

Tools used:
- tfsec
- Checkov
- Terrascan
- KICS

The goal was to compare tool effectiveness and identify critical
infrastructure security issues.

## Task 1 - Terraform & Pulumi Security Scanning

### Terraform Scanning Commands

#### tfsec

``` bash
docker run --rm -v "$(pwd)/labs/lab6/vulnerable-iac/terraform":/src   aquasec/tfsec:latest /src   --format json > labs/lab6/analysis/tfsec-results.json
```

#### Checkov

``` bash
docker run --rm -v "$(pwd)/labs/lab6/vulnerable-iac/terraform":/tf   bridgecrew/checkov:latest   -d /tf --framework terraform   -o json > labs/lab6/analysis/checkov-terraform-results.json
```

#### Terrascan

``` bash
docker run --rm -v "$(pwd)/labs/lab6/vulnerable-iac/terraform":/iac   tenable/terrascan:latest scan   -i terraform -d /iac   -o json > labs/lab6/analysis/terrascan-results.json
```

#### Terraform Tool Comparison

| Tool | Findings |
|---|---|
| tfsec | **53** |
| Checkov | **78** |
| Terrascan | **22** |


#### Terraform Security Issues Identified

Common issues detected by scanners:

-   Public S3 buckets
-   Security groups allowing access from 0.0.0.0/0
-   Unencrypted databases
-   Overly permissive IAM policies
-   Hardcoded credentials

These misconfigurations could lead to unauthorized access or data
exposure.

### Pulumi Security Analysis (KICS)

#### Scan Command

``` bash
docker run -t --rm -v "$(pwd)/labs/lab6/vulnerable-iac/pulumi":/src   checkmarx/kics:latest   scan -p /src -o /src/kics-report --report-formats json,html
```

#### Findings

| Severity | Count |
|---|---|
| High | **2** |
| Medium | **1** |
| Low | **0** |
| Total | **6** |

## Task 2 - Ansible Security Scanning

### Scan Command

``` bash
docker run -t --rm -v "$(pwd)/labs/lab6/vulnerable-iac/ansible":/src   checkmarx/kics:latest   scan -p /src -o /src/kics-report --report-formats json,html
```

### Findings

| Severity | Count |
|---|---|
| High | **0** |
| Medium | **0** |
| Low | **1** |
| Total | **10** |

## Task 3 - Tool Comparison and Security Insights

### Tool Comparison Matrix
| Criterion | tfsec | Checkov | Terrascan | KICS |
|---|---|---|---|---|
| Scan speed | Fast | Medium | Medium | Medium |
| Ease of use | High | Medium | Medium | Medium |
| Report quality | Good | Very good | Good | Very good |
| IaC support | Terraform | Multiple | Multiple | Multiple |
| CI/CD integration | Easy | Easy | Medium | Easy |

## Top 5 Critical Security Findings

1.  Public S3 buckets without access restrictions
2.  Security groups allowing inbound traffic from 0.0.0.0/0
3.  Unencrypted database storage
4.  Hardcoded credentials in infrastructure definitions
5.  Overly permissive IAM policies

## CI/CD Integration Strategy

Recommended approach:

1.  Run tfsec during pull request validation
2.  Run Checkov during CI builds
3.  Run KICS for multi-framework infrastructure checks
4.  Block deployment if high severity issues are detected

## Conclusion

This lab demonstrated the importance of Infrastructure-as-Code security
scanning.

Using multiple scanners improves detection coverage and helps prevent
insecure infrastructure deployments.

IaC security checks should be integrated into CI/CD pipelines to detect
vulnerabilities early.