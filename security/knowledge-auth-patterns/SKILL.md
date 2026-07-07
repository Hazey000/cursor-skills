---
name: knowledge-auth-patterns
description: >-
  Authentication and authorization patterns with trade-offs, implementation
  guidance, and common mistakes. Covers sessions, JWTs, OAuth/OIDC, API keys,
  mTLS, RBAC, ABAC, and ReBAC. Use when designing auth systems, reviewing
  auth implementations, or choosing between auth approaches.
---

# Authentication & Authorization Patterns

## Authentication Patterns

### Trade-offs Matrix

| Pattern | Stateless | Revocable | Scalable | Complexity | Best For |
|---------|-----------|-----------|----------|------------|----------|
| Session-based | No | Immediate | Needs shared store | Low | Traditional web apps |
| JWT (access token) | Yes | Hard (short TTL) | Excellent | Medium | APIs, microservices |
| OAuth 2.0 / OIDC | Yes | Via provider | Excellent | High | Third-party login, SSO |
| API keys | Yes | Immediate (DB) | Good | Low | Service-to-service, public APIs |
| mTLS | Yes | Via CRL/OCSP | Good | High | Service mesh, zero-trust |

### Session-Based Auth

```python
# Server creates session on login
session_id = secrets.token_urlsafe(32)
redis.setex(f"session:{session_id}", ttl=3600, value=json.dumps(user_data))
response.set_cookie("session_id", session_id, httponly=True, secure=True, samesite="Lax")
```

**Key decisions**:
- Store in Redis/Memcached for multi-instance deployments
- Set `HttpOnly`, `Secure`, `SameSite=Lax|Strict` on cookies
- Regenerate session ID after authentication (prevent fixation)
- Absolute timeout (e.g., 8h) + idle timeout (e.g., 30min)

### JWT (JSON Web Tokens)

```javascript
// Access token: short-lived, carries claims
const accessToken = jwt.sign(
  { sub: user.id, roles: user.roles, aud: "api.example.com" },
  privateKey,
  { algorithm: "RS256", expiresIn: "15m" }
);

// Verification: ALWAYS validate these
const decoded = jwt.verify(token, publicKey, {
  algorithms: ["RS256"],     // Restrict algorithms explicitly
  audience: "api.example.com",
  issuer: "auth.example.com"
});
```

**Critical rules**:
- Always validate: signature, `exp`, `iss`, `aud`, `alg`
- Never accept `alg: none`
- Keep access tokens short (5–15 min); use refresh tokens for longevity
- Don't store sensitive data in payload (it's base64, not encrypted)
- Use asymmetric keys (RS256/ES256) when multiple services verify

### OAuth 2.0 / OIDC

**Flow selection**:
| Client Type | Flow | Reason |
|-------------|------|--------|
| Server-side web app | Authorization Code | Can keep client_secret |
| SPA / mobile | Authorization Code + PKCE | No secret storage |
| Machine-to-machine | Client Credentials | No user interaction |
| NEVER use | Implicit | Deprecated; tokens in URL fragment |

**PKCE implementation** (for public clients):
1. Generate `code_verifier` (43–128 chars, cryptographic random)
2. Compute `code_challenge` = Base64URL(SHA256(code_verifier))
3. Send `code_challenge` + `code_challenge_method=S256` in auth request
4. Send `code_verifier` in token exchange

### API Keys

- Generate with high entropy (≥32 bytes, cryptographic random)
- Store hashed (SHA-256 is fine; no need for bcrypt since keys are high-entropy)
- Prefix keys for identifiability: `sk_live_`, `pk_test_`
- Support key rotation: allow multiple active keys per client
- Scope keys to specific permissions/resources

### mTLS (Mutual TLS)

- Both client and server present certificates
- Use for service-to-service in zero-trust networks
- Manage via service mesh (Istio, Linkerd) or cert-manager
- Short-lived certs (24h) with auto-rotation reduce revocation complexity

## Token Storage

| Storage Location | XSS Safe | CSRF Safe | Use Case |
|-----------------|-----------|-----------|----------|
| HttpOnly cookie | Yes | Need CSRF token | Session-based web apps |
| Memory (JS variable) | Partial | Yes | SPAs (lost on refresh) |
| sessionStorage | No | Yes | Short-lived SPA tokens |
| localStorage | No | Yes | NEVER for sensitive tokens |

**Recommended SPA pattern**: Store refresh token in HttpOnly cookie; keep access token in memory only. On page reload, silently refresh.

## Refresh Token Flow

```
Client → POST /token (grant_type=refresh_token, refresh_token=RT1)
Server:
  1. Validate RT1 exists and is not revoked
  2. Issue new access token AT2
  3. Issue new refresh token RT2 (rotation)
  4. Invalidate RT1
  5. Return AT2 + RT2
```

**Refresh token rotation**: Issue a new refresh token with every use. If a refresh token is used twice, revoke the entire family (indicates theft).

## Authorization Patterns

### RBAC (Role-Based Access Control)

```python
# Simple: user has roles, roles have permissions
ROLE_PERMISSIONS = {
    "viewer": {"read"},
    "editor": {"read", "write"},
    "admin": {"read", "write", "delete", "manage_users"},
}

def check_permission(user, permission):
    return any(permission in ROLE_PERMISSIONS[role] for role in user.roles)
```

**When to use**: Permission model is relatively static; roles map cleanly to job functions.

**Limitations**: Role explosion when permissions are fine-grained or context-dependent.

### ABAC (Attribute-Based Access Control)

Decisions based on attributes of user, resource, action, and environment:

```python
def check_access(user, resource, action, context):
    policies = get_applicable_policies(resource.type, action)
    for policy in policies:
        if policy.evaluate(user.attributes, resource.attributes, context):
            return policy.effect  # ALLOW or DENY
    return DENY  # Default deny
```

**When to use**: Complex rules involving time, location, resource properties, or relationships.

**Example policies**:
- "Doctors can view patient records only in their department during working hours"
- "Users can edit documents they created or that are shared with their team"

### ReBAC (Relationship-Based Access Control)

Authorization based on relationships between entities (like Google Zanzibar / OpenFGA):

```
// Relationship tuples
document:readme#viewer@user:alice
document:readme#owner@user:bob
folder:docs#viewer@group:engineering#member

// Check: can alice view document:readme?
// Traverse: alice → viewer of readme → YES
```

**When to use**: Social/collaborative apps, multi-tenant platforms, any system where sharing/delegation is core.

**Implementations**: OpenFGA, SpiceDB, Ory Keto.

## Common Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| JWT secret in env without rotation | Key compromise = full auth bypass | Use asymmetric keys; automate rotation |
| Checking permissions in UI only | API is unprotected | Enforce server-side on every request |
| Long-lived tokens without revocation | Stolen token usable indefinitely | Short access tokens + refresh rotation |
| Storing bcrypt-hashed API keys | Slow lookups at scale | SHA-256 is sufficient for high-entropy keys |
| No re-authentication for sensitive ops | Session hijack escalates easily | Require password/MFA for password change, payment, etc. |
| Mixing authn and authz logic | Hard to audit, easy to bypass | Separate middleware: authenticate → authorize → handle |
| Trusting client-supplied user ID | IDOR attacks | Derive user identity from verified token server-side |

## Session Management Checklist

- [ ] Session ID generated server-side with ≥128 bits entropy
- [ ] Regenerated after authentication state changes
- [ ] Absolute timeout enforced (max session lifetime)
- [ ] Idle timeout enforced (inactivity expiry)
- [ ] Invalidated server-side on logout (not just client cookie deletion)
- [ ] Bound to additional context (user-agent, IP range) for anomaly detection
- [ ] Concurrent session limits enforced where appropriate

## Cross-References

- **`knowledge-owasp-top-10`** — A07 (Identification and Authentication Failures)
- **`knowledge-secure-by-design`** — Fail securely, least privilege applied to auth
