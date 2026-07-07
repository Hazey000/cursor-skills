---
name: knowledge-fail-fast
description: >-
  Fail-fast philosophy, error propagation strategy, and early validation
  principles. Covers preconditions, assertions, input validation boundaries,
  and how to make errors visible immediately. Use when designing error handling,
  validation layers, or debugging silent failures.
---

# Fail Fast

## Purpose

Systems that fail fast are easier to debug, more reliable, and less likely to corrupt data. This skill covers when and how to apply fail-fast principles.

## Core Principle

**Detect errors at the earliest possible point and surface them immediately** rather than allowing them to propagate silently through the system.

## Where to Fail Fast

| Boundary | What to validate | How to fail |
|----------|-----------------|-------------|
| Public API entry | Request format, required fields, types | 400 Bad Request with specifics |
| Service boundary | Preconditions, invariants | Throw/return typed error |
| Configuration load | Required config, valid ranges | Crash at startup |
| Constructor/factory | Required dependencies, valid state | Throw immediately |
| Data ingestion | Schema compliance, referential integrity | Reject with reason |

## Fail-Fast Patterns

### Preconditions (Guard Clauses)
Validate assumptions at function entry. Return/throw immediately if violated.

### Constructor Validation
Objects should never exist in an invalid state. Validate in the constructor.

### Startup Checks
Verify all required configuration, connections, and dependencies at startup — not at first use.

### Schema Validation at Boundaries
Parse and validate external data (API requests, file imports, message queue payloads) at the system boundary. Work with validated types internally.

## When NOT to Fail Fast

- **Resilient consumers**: Background workers that should retry on transient errors
- **User-facing bulk operations**: Collect all validation errors, don't stop at first
- **Distributed systems**: Circuit breakers and graceful degradation over hard failures
- **Data pipelines**: Dead-letter queue over crash (log and continue for partial failures)

## Error Propagation Strategy

1. **Boundaries**: Validate and transform to typed errors
2. **Domain logic**: Propagate typed errors, don't catch-and-ignore
3. **Infrastructure**: Wrap with context, preserve stack trace
4. **Top-level handler**: Log, respond appropriately, never swallow

**Anti-patterns**:
- Catching exceptions and returning null/default values
- Empty catch blocks
- Logging an error but continuing as if nothing happened
- Returning error codes that callers never check

## Assertions vs Validation

| Concern | Use | Example |
|---------|-----|---------|
| Validation | External input you don't control | User request, API response |
| Assertion | Internal invariants that should never be violated | "This list should never be empty here" |

Assertions indicate programmer errors. Validation handles expected invalid input.

## Related Skills

- `principles/knowledge-defensive-programming` — complementary approach
- `backend/knowledge-error-handling` — error hierarchies and recovery
- `security/knowledge-owasp-top-10` — input validation as security control
