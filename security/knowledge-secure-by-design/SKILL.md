---
name: knowledge-secure-by-design
description: >-
  Security as an architectural property — principles, patterns, and decision
  frameworks for building systems that are secure by construction rather than
  by bolt-on controls. Use when designing systems, reviewing architecture, or
  making security trade-off decisions.
---

# Secure by Design

Security is a quality attribute of architecture, not a feature you add later. These principles reduce attack surface structurally.

## Core Principles

### 1. Least Privilege

Grant the minimum access required for a function to operate.

**Apply at every layer**:
- OS: run processes as non-root with minimal capabilities
- Database: separate read/write/admin accounts per service
- Cloud IAM: scope roles to specific resources, not `*`
- Application: default deny; grant permissions explicitly
- API: return only the fields the caller needs

**Test**: "If this component is compromised, what can the attacker reach?" Minimize that blast radius.

### 2. Defense in Depth

No single control is sufficient. Layer independent defenses so a failure in one doesn't mean total compromise.

**Example layers for a web API**:
1. WAF / rate limiter (network edge)
2. Input validation (application boundary)
3. Parameterized queries (data access layer)
4. Column-level encryption (data layer)
5. Monitoring and alerting (detection layer)

**Anti-pattern**: Relying solely on a WAF to prevent injection.

### 3. Fail Securely

When a component fails, it should deny access rather than grant it.

```python
# WRONG: fail-open
def check_permission(user, resource):
    try:
        return authz_service.check(user, resource)
    except AuthzServiceError:
        return True  # "Let them through, service is down"

# RIGHT: fail-closed
def check_permission(user, resource):
    try:
        return authz_service.check(user, resource)
    except AuthzServiceError:
        log.error("Authz service unavailable", exc_info=True)
        return False
```

**Corollaries**:
- Default deny in all access control
- Return generic errors to clients; log details server-side
- Crash rather than continue in an inconsistent security state

### 4. Don't Trust Services

Treat every external system (and internal microservice) as potentially compromised.

- Validate data received from other services, not just user input
- Use mutual TLS (mTLS) between services
- Apply rate limits on internal APIs (lateral movement defense)
- Verify JWTs/tokens from other services; don't just trust headers

### 5. Separation of Duties

No single actor or component should control an entire critical operation.

- Separate CI/CD build from deploy; require approval for production
- Split admin actions into propose + approve (4-eyes principle)
- Isolate secrets management from application runtime
- Use different credentials for read vs. write operations

### 6. Avoid Security by Obscurity

Assume the attacker knows your architecture. Security must not depend on hidden implementation details.

- Open-source your algorithms; rely on key secrecy not algorithm secrecy
- Don't rely on non-standard ports, hidden URLs, or renamed admin panels
- Treat internal network as hostile (zero-trust)

**Obscurity is acceptable as ONE additional layer** (e.g., hiding version headers reduces drive-by scans) but never as the primary control.

## Architectural Patterns

### Trust Boundaries

Identify where data crosses trust levels. Every boundary needs:
- Authentication (who are you?)
- Authorization (are you allowed?)
- Input validation (is this data safe?)
- Logging (what happened?)

Common trust boundaries:
- Client → Server
- Public subnet → Private subnet
- Service A → Service B
- Application → Database
- Your system → Third-party API

### Secure Defaults

Design APIs and configurations so the default state is secure:

```go
// Secure default: TLS required, opt-out must be explicit
type ClientConfig struct {
    TLSEnabled  bool   // default true via constructor
    TLSMinVersion uint16 // default tls.VersionTLS12
}

func NewClientConfig() ClientConfig {
    return ClientConfig{TLSEnabled: true, TLSMinVersion: tls.VersionTLS12}
}
```

### Blast Radius Containment

Architect so a breach in one component cannot cascade:
- Network segmentation (VPCs, security groups)
- Separate databases per service (no shared-DB microservices)
- Short-lived credentials (rotate < breach detection time)
- Feature flags to disable compromised functionality

## Security Trade-off Decision Framework

When security conflicts with usability, performance, or delivery speed:

### Step 1: Classify the Risk

| Impact | Description | Examples |
|--------|-------------|----------|
| Critical | Data breach, regulatory fine, service destruction | PII leak, auth bypass |
| High | Significant data exposure, service degradation | Internal API exposure |
| Medium | Limited exposure, contained impact | Info disclosure in errors |
| Low | Minimal impact, no sensitive data | CSS injection |

### Step 2: Assess the Trade-off

Ask:
1. **What is the threat?** (specific attack, not vague "security")
2. **What is the probability?** (exposed to internet vs internal tool)
3. **What is the cost of mitigation?** (development time, UX friction, performance)
4. **What is the cost of NOT mitigating?** (breach impact × probability)
5. **Can we mitigate later without rearchitecting?** (if yes, might defer; if no, do it now)

### Step 3: Document the Decision

If accepting risk:
- Record it as a known risk with owner and review date
- Set a calendar reminder to reassess
- Ensure monitoring covers the accepted risk scenario

If mitigating:
- Choose the approach with lowest ongoing maintenance burden
- Prefer structural fixes (make it impossible) over procedural fixes (make it detectable)

## When to Push Back

Always push back (non-negotiable):
- Storing plaintext passwords
- Disabling TLS in production
- Hardcoding secrets in source
- Running as root without justification
- Skipping auth on data-mutating endpoints

Negotiate (trade-offs exist):
- Level of input validation strictness
- Session timeout duration
- MFA requirements for low-risk operations
- Logging verbosity vs. performance

## Cross-References

- **`knowledge-owasp-top-10`** — Specific vulnerability categories this prevents
- **`knowledge-privacy-by-design`** — Privacy as a parallel architectural concern
