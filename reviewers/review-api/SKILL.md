---
name: review-api
description: >-
  Evaluate API design for consistency, correct HTTP semantics, error handling,
  pagination, versioning, security, and documentation quality. Covers REST
  and GraphQL.
---

# API Design Review

Evaluate API design for correctness, consistency, usability, and security.

**Cross-reference**: `api/knowledge-rest-principles` for REST constraints and resource modeling.

## When to Use

- Before merging new or modified API endpoints
- When reviewing OpenAPI/Swagger specs or GraphQL schemas
- When evaluating API backward compatibility
- During API design reviews before implementation

## Severity Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Breaking change, security hole, or correctness issue — fix before merge |
| **WARNING** | Inconsistency or poor usability that will cause client developer friction — fix soon |
| **SUGGESTION** | Polish that improves developer experience — consider for next iteration |

## Review Checklist

### 1. Resource Design

| Check | Severity if violated |
|-------|---------------------|
| Resources are nouns, not verbs (`/users`, not `/getUsers`) | WARNING |
| Consistent pluralization (`/users`, `/orders`) | WARNING |
| Nested resources reflect real ownership (`/users/{id}/orders`) | WARNING |
| Nesting doesn't exceed 2 levels (`/a/{id}/b` max, not `/a/{id}/b/{id}/c/{id}/d`) | WARNING |
| No resource naming collisions or ambiguity | WARNING |
| Collection and item endpoints are distinct | WARNING |

### 2. HTTP Semantics

| Check | Severity if violated |
|-------|---------------------|
| GET is safe and idempotent (no side effects) | CRITICAL |
| POST used for creation, returns 201 with Location header | WARNING |
| PUT replaces entire resource, PATCH for partial update | WARNING |
| DELETE is idempotent (second call returns 204 or 404, no error) | WARNING |
| No state mutation via GET or query parameters | CRITICAL |
| Correct status codes: 200/201/204 for success, 4xx for client error, 5xx for server error | WARNING |

**Status code quick reference**:

| Code | When to use |
|------|------------|
| 200 | Successful GET, PUT, PATCH |
| 201 | Successful POST (resource created) |
| 204 | Successful DELETE or action with no body |
| 400 | Malformed request / validation failure |
| 401 | Not authenticated |
| 403 | Authenticated but not authorized |
| 404 | Resource doesn't exist |
| 409 | Conflict (duplicate, state violation) |
| 422 | Semantically invalid (valid syntax, bad data) |
| 429 | Rate limited |
| 500 | Unexpected server error |

### 3. Error Handling

| Check | Severity if violated |
|-------|---------------------|
| Errors use a consistent envelope format | WARNING |
| Error response includes: code, message, and optionally field-level details | WARNING |
| Error messages are actionable (not just "Bad Request") | WARNING |
| No stack traces or internal details leaked in production errors | CRITICAL |
| Validation errors return all failures, not just the first | SUGGESTION |

**Standard error format**:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      { "field": "email", "message": "Must be a valid email address" }
    ]
  }
}
```

### 4. Pagination, Filtering, and Sorting

| Check | Severity if violated |
|-------|---------------------|
| Collection endpoints are paginated (no unbounded lists) | CRITICAL |
| Pagination style is consistent (cursor-based preferred for large/real-time data) | WARNING |
| Page metadata included (total count or has_next indicator) | WARNING |
| Filtering uses query parameters (`?status=active`) | WARNING |
| Sorting is explicit (`?sort=created_at&order=desc`) | SUGGESTION |
| Default page size is reasonable and documented | SUGGESTION |

### 5. Versioning

| Check | Severity if violated |
|-------|---------------------|
| Versioning strategy exists (URL prefix, header, or content negotiation) | WARNING |
| Breaking changes increment the version | CRITICAL |
| Deprecated endpoints have sunset dates communicated | WARNING |
| Multiple versions can coexist during migration | SUGGESTION |

### 6. Security

| Check | Severity if violated |
|-------|---------------------|
| Authentication required on all non-public endpoints | CRITICAL |
| Authorization checked at resource level, not just endpoint | CRITICAL |
| Rate limiting configured | WARNING |
| Input validated and sanitized | CRITICAL |
| CORS configured restrictively | WARNING |
| No sensitive data in URLs (tokens, passwords in query strings) | CRITICAL |

### 7. Backward Compatibility

| Check | Severity if violated |
|-------|---------------------|
| New fields are optional (existing clients don't break) | CRITICAL |
| Removed fields go through deprecation cycle | CRITICAL |
| Enum values are only added, not removed or renamed | CRITICAL |
| Response shape changes are additive only | CRITICAL |
| URL/path changes maintain old routes as redirects or aliases | WARNING |

### 8. Documentation

| Check | Severity if violated |
|-------|---------------------|
| All endpoints documented with request/response examples | WARNING |
| Auth requirements documented per endpoint | WARNING |
| Error codes and their meaning documented | WARNING |
| Rate limits documented | SUGGESTION |
| OpenAPI/Swagger spec is up to date with implementation | WARNING |
| Changelog maintained for API changes | SUGGESTION |

## Output Format

```markdown
## API Review: [API/Endpoint Name]

**Scope**: [Endpoints reviewed]
**Overall**: [PASS | PASS WITH WARNINGS | FAIL]

### Findings

#### Design
##### [WARNING] Verb in resource name
**Location**: `POST /api/createUser`
**Issue**: Resource uses verb instead of noun
**Fix**: `POST /api/users` — the HTTP method implies creation

#### Security
##### [CRITICAL] Missing auth on admin endpoint
**Location**: `GET /api/admin/users`
**Issue**: No authentication middleware
**Fix**: Add auth middleware and admin role check

...

### Summary
| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Design | 0 | 2 | 1 |
| HTTP Semantics | 0 | 1 | 0 |
| Error Handling | 0 | 1 | 0 |
| Pagination | 1 | 0 | 1 |
| Security | 1 | 1 | 0 |
| Compatibility | 0 | 0 | 0 |
| Documentation | 0 | 2 | 0 |
| **Total** | **2** | **7** | **2** |

### Recommendations
1. <Prioritized by impact>
2. ...
```
