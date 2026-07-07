---
name: knowledge-error-handling
description: >-
  Error handling strategy covering error hierarchies, propagation patterns,
  recovery strategies, and consistent error responses. Use when designing
  error handling for services, choosing between exceptions and result types,
  or standardizing error responses.
---

# Error Handling

## Purpose

Design consistent, debuggable error handling that distinguishes recoverable from fatal errors.

## Error Classification

| Category | Recoverable? | Action | Example |
|----------|-------------|--------|---------|
| Validation | Yes (user fixes) | Return specific feedback | Invalid email format |
| Business rule | Yes (change request) | Return domain error | Insufficient funds |
| Transient | Yes (retry) | Retry with backoff | Network timeout |
| Permanent external | No (for this request) | Fail gracefully, alert | Third-party API down |
| Programming error | No | Crash, fix code | Null pointer, assertion failure |

## Error Propagation Strategy

1. **Catch at boundaries, not in business logic**
2. **Add context as errors propagate** (wrap, don't replace)
3. **Translate at layer boundaries** (domain error → HTTP error → client error)
4. **Never swallow errors silently**

## Error Response Design

Consistent error format for APIs:

```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Human-readable description",
    "details": [
      {"field": "email", "issue": "Invalid email format"}
    ],
    "request_id": "req_abc123"
  }
}
```

**Rules**:
- Machine-readable code (for client logic)
- Human-readable message (for debugging)
- Structured details (for field-level errors)
- Request ID (for log correlation)
- Never expose stack traces or internal details to clients

## Decision: Exceptions vs Result Types

| Use exceptions when | Use result types when |
|--------------------|--------------------|
| Truly exceptional (shouldn't happen) | Expected failure modes |
| Language/framework convention | Functional programming style |
| Error handling is centralized | Caller must handle each case |
| Performance isn't critical | Hot path |

## Anti-Patterns

- Empty catch blocks
- Catching and rethrowing without adding context
- Using exceptions for flow control
- Generic error messages ("Something went wrong")
- Logging error but continuing as if nothing happened
- Returning null instead of an error

## Related Skills

- `principles/knowledge-fail-fast` — fail fast at boundaries
- `api/execution-rest-api-design` — error response design
- `observability/execution-logging-strategy` — logging errors with context
