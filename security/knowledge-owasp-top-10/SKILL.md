---
name: knowledge-owasp-top-10
description: >-
  OWASP Top 10 vulnerability categories with code-level examples and specific
  mitigations. Use when reviewing code for security issues, designing secure
  APIs, or advising on common web application vulnerabilities.
---

# OWASP Top 10 (2021)

Practical reference for the most critical web application security risks. Focus on prevention at the code level.

## A01: Broken Access Control

**What**: Users act outside intended permissions—viewing other users' data, modifying access levels, accessing admin pages.

**Code examples**:
```python
# VULNERABLE: Direct object reference without ownership check
@app.get("/api/invoices/{invoice_id}")
def get_invoice(invoice_id: int, user: User):
    return db.query(Invoice).get(invoice_id)  # No check that user owns this

# FIXED
def get_invoice(invoice_id: int, user: User):
    invoice = db.query(Invoice).filter(Invoice.id == invoice_id, Invoice.owner_id == user.id).first()
    if not invoice:
        raise HTTPException(404)
    return invoice
```

**Mitigations**:
- Deny by default; enforce ownership checks on every data access
- Disable directory listing; remove `.git`, backup files from webroot
- Rate-limit APIs to minimize automated scanning
- Invalidate JWT/sessions on logout; use short-lived tokens
- Unit test access control for every endpoint (positive AND negative cases)

## A02: Cryptographic Failures

**What**: Exposure of sensitive data through weak/missing encryption, plaintext storage, deprecated algorithms.

**Common mistakes**:
- Storing passwords with MD5/SHA1 instead of bcrypt/argon2
- Transmitting sensitive data over HTTP
- Hardcoding encryption keys in source
- Using ECB mode, small key sizes, or no IV/nonce

**Mitigations**:
- Classify data by sensitivity; encrypt PII at rest and in transit
- Use TLS 1.2+ everywhere; enforce HSTS
- Passwords: argon2id or bcrypt with cost factor ≥12
- Keys: manage via secrets manager (Vault, AWS KMS), never in code
- Disable caching on responses containing sensitive data

## A03: Injection

**What**: Untrusted data sent to an interpreter as part of a command—SQL, NoSQL, OS, LDAP, XPath.

**Code examples**:
```javascript
// VULNERABLE: String concatenation in SQL
const query = `SELECT * FROM users WHERE id = '${req.params.id}'`;

// FIXED: Parameterized query
const query = 'SELECT * FROM users WHERE id = $1';
const result = await db.query(query, [req.params.id]);
```

**Mitigations**:
- Use parameterized queries / prepared statements for ALL database access
- Use ORM methods that auto-parameterize; avoid raw query escape hatches
- Validate and sanitize input: allowlist > denylist
- Escape output contextually (HTML, JS, URL, CSS, SQL)
- Limit database account privileges (read-only where possible)

## A04: Insecure Design

**What**: Missing or ineffective security controls at the design level—no threat model, no abuse case analysis.

**Mitigations**:
- Threat model during design (see `execution-threat-modeling`)
- Write abuse cases alongside use cases
- Apply secure design patterns: trust boundaries, rate limiting, circuit breakers
- Segregate tenants at the data layer, not just the application layer
- Reference architecture reviews for security-critical flows

## A05: Security Misconfiguration

**What**: Default credentials, unnecessary features enabled, overly permissive cloud IAM, missing security headers, verbose errors in production.

**Mitigations**:
- Automate hardening: infrastructure as code with security baselines
- Remove unused features, frameworks, endpoints, sample apps
- Review cloud IAM quarterly; follow least privilege
- Security headers: `Content-Security-Policy`, `X-Content-Type-Options`, `Strict-Transport-Security`, `X-Frame-Options`
- Disable stack traces / debug info in production error responses

## A06: Vulnerable and Outdated Components

**What**: Using libraries, frameworks, or OS components with known CVEs.

**Mitigations**:
- Automate dependency scanning (Dependabot, Snyk, Trivy) in CI
- Pin dependency versions; review changelogs before upgrading
- Remove unused dependencies
- Subscribe to security advisories for critical dependencies
- Use SBOM generation for production images

## A07: Identification and Authentication Failures

**What**: Weak passwords allowed, credential stuffing not mitigated, session IDs in URLs, missing MFA.

**Mitigations**:
- Enforce password complexity + check against breach databases (haveibeenpwned API)
- Implement MFA for privileged operations
- Rate-limit and lockout on login failures (progressive delays)
- Generate session IDs server-side with high entropy; invalidate on auth state changes
- Never expose session tokens in URLs or logs
- See `knowledge-auth-patterns` for implementation details

## A08: Software and Data Integrity Failures

**What**: CI/CD pipeline compromises, unsigned updates, deserialization of untrusted data.

**Mitigations**:
- Verify digital signatures on updates and dependencies
- Use lock files and verify checksums in CI
- Restrict CI/CD pipeline permissions; require approval for production deploys
- Avoid deserializing untrusted data; if required, use allowlists for types
- Separate build and deploy credentials

## A09: Security Logging and Monitoring Failures

**What**: Attacks go undetected due to insufficient logging, no alerting, or logs that aren't monitored.

**Mitigations**:
- Log all authentication events (success + failure), access control failures, input validation failures
- Use structured logging with correlation IDs; never log secrets or full PII
- Alert on anomalous patterns: brute force, privilege escalation attempts, unusual data access volumes
- Ensure logs are tamper-evident (append-only, shipped to central SIEM)
- Test your alerting: run game days / red team exercises

## A10: Server-Side Request Forgery (SSRF)

**What**: Application fetches a URL supplied by the user without validation, allowing access to internal services.

**Code examples**:
```python
# VULNERABLE
resp = requests.get(user_supplied_url)  # Can hit http://169.254.169.254/metadata

# FIXED: Allowlist + DNS resolution check
from urllib.parse import urlparse
import ipaddress

def safe_fetch(url: str):
    parsed = urlparse(url)
    if parsed.scheme not in ('http', 'https'):
        raise ValueError("Invalid scheme")
    resolved = socket.getaddrinfo(parsed.hostname, None)[0][4][0]
    ip = ipaddress.ip_address(resolved)
    if ip.is_private or ip.is_loopback or ip.is_link_local:
        raise ValueError("Internal address blocked")
    return requests.get(url, allow_redirects=False, timeout=5)
```

**Mitigations**:
- Validate and sanitize all user-supplied URLs
- Allowlist permitted schemes, ports, and destinations
- Block access to metadata endpoints (169.254.169.254) at network layer
- Use a dedicated egress proxy for outbound requests
- Disable HTTP redirects or re-validate after each redirect

## Quick Decision Guide

| Reviewing... | Check for... |
|---|---|
| API endpoint | A01 (authz), A03 (injection), A07 (authn) |
| Data storage | A02 (encryption), A09 (logging) |
| Third-party integration | A06 (CVEs), A08 (integrity), A10 (SSRF) |
| Infrastructure/config | A05 (misconfiguration) |
| New feature design | A04 (insecure design) |

## Cross-References

- **`knowledge-secure-by-design`** — Architectural principles that prevent entire categories
- **`knowledge-auth-patterns`** — Deep dive on A07 authentication/authorization
