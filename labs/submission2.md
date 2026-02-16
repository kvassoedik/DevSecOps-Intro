## System Overview

The modeled system is OWASP Juice Shop running locally in a Docker container. Users access the application through a web browser either directly via HTTP on localhost:3000 or through an optional reverse proxy.

Application data such as the SQLite database, uploads, and logs are stored on a host-mounted volume. The architecture includes three main trust boundaries: Internet, Host, and Container Network.

## Baseline Risk Overview

The threat model identified 23 risks across 15 categories.
No Critical or High risks were found, but the system contains:

- 4 Elevated risks
- 14 Medium risks
- 5 Low risks

This indicates a moderate risk posture typical for an intentionally vulnerable training application.

## Top 5 Risks

| Risk | Severity | Likelihood | Impact | Score |
|------|----------|------------|--------|--------|
| XSS | Elevated | Likely | Medium | 432 |
| Missing Authentication | Elevated | Likely | Medium | 432 |
| Unencrypted Communication | Elevated | Likely | High | 433 |
| CSRF | Medium | Very Likely | Low | 241 |
| SSRF | Medium | Likely | Low | 231 |

## Architectural Observations

The most attractive attack target is Persistent Storage (RAA 100%), followed by the Juice Shop Application (RAA 70%). This shows attackers are primarily interested in stored data rather than the frontend itself.

## Secure Variant Changes

The secure model enforced HTTPS communication and encryption of stored data.
This directly mitigates risks related to unencrypted communication and information disclosure.
