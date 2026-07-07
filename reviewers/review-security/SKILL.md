---
name: review-security
description: >-
  Identify security vulnerabilities, auth weaknesses, injection risks, secrets
  exposure, and access control issues. Categorizes findings by OWASP Top 10.
  Produces a severity-rated security review report.
---

# Security Review

Evaluate code for security vulnerabilities, focusing on the OWASP Top 10 and secure coding practices.

**Cross-reference**: `security/knowledge-owasp-top-10` for vulnerability details and mitigations.

## When to Use

- Before merging code that handles auth, user input, or sensitive data
- When adding new API endpoints or external integrations
- When changing infrastructure, deployment, or secrets configuration
- Periodic security audit of existing code

## Severity Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Exploitable vulnerability — block merge |
| **WARNING** | Security weakness that increases attack surface — fix before production |
| **SUGGESTION** | Defense-in-depth improvement — implement when practical |

## Review Checklist by OWASP Category

### A01: Broken Access Control

| Check | Severity if violated |
|-------|---------------------|
| Every endpoint enforces authentication | CRITICAL |
| Authorization checks use deny-by-default | CRITICAL |
| No IDOR (Insecure Direct Object References) — users can't access others' resources by changing IDs | CRITICAL |
| CORS is restrictive, not `Access-Control-Allow-Origin: *` | WARNING |
| Rate limiting on sensitive endpoints (login, password reset, API) | WARNING |
| Admin functions are not accessible by guessing URLs | CRITICAL |
| JWT/session tokens are validated on every request, not just at login | CRITICAL |

### A02: Cryptographic Failures

| Check | Severity if violated |
|-------|---------------------|
| Passwords hashed with bcrypt/scrypt/argon2, never MD5/SHA1 | CRITICAL |
| Sensitive data encrypted at rest and in transit (TLS) | CRITICAL |
| No secrets in source code, logs, or error messages | CRITICAL |
| Encryption keys are rotatable and not hardcoded | WARNING |
| Sensitive data not stored in JWT payload | WARNING |

### A03: Injection

| Check | Severity if violated |
|-------|---------------------|
| SQL queries use parameterized statements, never string concatenation | CRITICAL |
| NoSQL queries are parameterized | CRITICAL |
| OS command execution uses allowlists, never user input directly | CRITICAL |
| Template rendering escapes user input (XSS prevention) | CRITICAL |
| LDAP, XML, and path traversal injection vectors are addressed | CRITICAL |

### A04: Insecure Design

| Check | Severity if violated |
|-------|---------------------|
| Threat model exists for security-critical flows | WARNING |
| Business logic abuse scenarios are considered (e.g., coupon reuse) | WARNING |
| Security requirements are explicit, not assumed | SUGGESTION |

### A05: Security Misconfiguration

| Check | Severity if violated |
|-------|---------------------|
| Debug mode / verbose errors disabled in production | CRITICAL |
| Default credentials changed | CRITICAL |
| Security headers set (CSP, X-Frame-Options, HSTS, X-Content-Type-Options) | WARNING |
| Unnecessary features/endpoints/ports disabled | WARNING |
| Directory listing disabled | WARNING |

### A06: Vulnerable Components

| Check | Severity if violated |
|-------|---------------------|
| Dependencies are pinned to specific versions | WARNING |
| No known CVEs in dependency tree | CRITICAL |
| Lock files committed and reviewed | WARNING |
| Unused dependencies removed | SUGGESTION |

### A07: Authentication Failures

| Check | Severity if violated |
|-------|---------------------|
| Multi-factor auth available for sensitive operations | SUGGESTION |
| Session tokens regenerated after login | WARNING |
| Session expiry and idle timeout configured | WARNING |
| Password complexity requirements enforced | WARNING |
| Brute-force protection (lockout/exponential backoff) | WARNING |

### A08: Data Integrity Failures

| Check | Severity if violated |
|-------|---------------------|
| CI/CD pipeline integrity (signed commits, protected branches) | WARNING |
| Deserialization of untrusted data is avoided or validated | CRITICAL |
| Software updates verified with signatures | SUGGESTION |

### A09: Logging and Monitoring Failures

| Check | Severity if violated |
|-------|---------------------|
| Auth events (login, failure, logout) are logged | WARNING |
| Logs don't contain sensitive data (passwords, tokens, PII) | CRITICAL |
| Tamper-evident logging in place | SUGGESTION |
| Alerting on suspicious patterns (repeated auth failures) | WARNING |

### A10: Server-Side Request Forgery (SSRF)

| Check | Severity if violated |
|-------|---------------------|
| User-supplied URLs are validated against an allowlist | CRITICAL |
| Internal network addresses blocked from user-controlled requests | CRITICAL |
| URL redirects validated | WARNING |

## Additional Checks

### Secrets Management

| Check | Severity |
|-------|----------|
| Secrets loaded from env vars or vault, never config files in repo | CRITICAL |
| `.env` / credentials files in `.gitignore` | CRITICAL |
| API keys scoped to minimum required permissions | WARNING |
| Secrets rotation process documented | SUGGESTION |

### Input Validation

| Check | Severity |
|-------|----------|
| All external input validated at system boundary | CRITICAL |
| Validation is allowlist-based (accept known good), not denylist | WARNING |
| File uploads validated (type, size, content) | CRITICAL |
| Request size limits enforced | WARNING |

## Output Format

```markdown
## Security Review: [Component/Service Name]

**Scope**: [What was reviewed]
**Overall**: [PASS | PASS WITH WARNINGS | FAIL]

### Findings by OWASP Category

#### A03: Injection
##### [CRITICAL] SQL Injection in User Search
**Location**: `src/users/search.ts:87`
**Issue**: User input concatenated into SQL query
**Exploit**: Attacker can extract/modify arbitrary data
**Fix**: Use parameterized query: `db.query('SELECT * FROM users WHERE name = $1', [input])`

...

### Summary
| OWASP Category | Critical | Warning | Suggestion |
|----------------|----------|---------|------------|
| A01: Broken Access Control | 0 | 1 | 0 |
| A03: Injection | 1 | 0 | 0 |
| ... | ... | ... | ... |
| **Total** | **N** | **N** | **N** |

### Recommendations
1. <Prioritized remediation>
2. ...
```
