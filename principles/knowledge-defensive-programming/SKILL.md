---
name: knowledge-defensive-programming
description: >-
  Defensive programming principles including input validation, invariant
  protection, safe defaults, and trust boundaries. Covers the spectrum from
  paranoid to pragmatic defense. Use when hardening code against unexpected
  inputs, integrating with external systems, or designing public APIs.
---

# Defensive Programming

## Purpose

Write code that protects itself from invalid inputs, violated assumptions, and unexpected states — without becoming paranoid boilerplate.

## Core Principle

**Don't trust data you don't control.** Validate at trust boundaries, assert internal invariants, and design safe defaults.

## Trust Boundaries

Code lives at different trust levels. Defend at the transitions:

| From | To | Defense needed |
|------|----|---------------|
| External user | Your API | Full input validation, rate limiting |
| External service | Your code | Schema validation, timeout, circuit breaker |
| Internal service | Internal service | Light validation, contract testing |
| Your code | Your code (same module) | Assertions for invariants only |

**Rule**: Defense intensity should match the trust gap.

## Techniques

### Safe Defaults
- Default to deny (permissions, feature flags)
- Default to closed (network rules, CORS)
- Default to minimal (data exposure, logging PII)

### Input Validation at Boundaries
- Parse, don't validate — transform raw input into validated types
- Whitelist over blacklist
- Validate structure AND semantics (e.g., email format AND domain exists)

### Immutability
- Prefer immutable data structures for shared state
- Return copies, not references to internal state
- Use `readonly`/`final`/`const` by default

### Null Safety
- Avoid returning null — use Option/Maybe types or empty collections
- If null is possible, handle it at the boundary, not throughout the code
- Never pass null as a meaningful parameter

### Timeouts and Limits
- Every external call needs a timeout
- Every unbounded collection needs a size limit
- Every user input needs a length limit
- Every retry needs a maximum count

## Pragmatic Defense (Avoiding Over-Defense)

Not everything needs maximum defense:

| Context | Appropriate level |
|---------|-------------------|
| Public API | Maximum: validate everything, assume malice |
| Internal module boundary | Medium: type safety + key invariants |
| Private method | Minimal: assertions for debugging only |
| Hot loop | None: trust validated data from outer boundary |

**Anti-pattern**: Defensive checks repeated at every layer (validate once at the boundary, trust internally).

## Defensive Design Checklist

- [ ] External inputs validated at entry point
- [ ] Null/empty cases handled explicitly
- [ ] External calls have timeouts
- [ ] Collections have size bounds
- [ ] Defaults are safe (deny, closed, minimal)
- [ ] Invariants asserted at construction time
- [ ] Errors fail visibly, not silently

## Related Skills

- `principles/knowledge-fail-fast` — complementary: fail fast + defend at boundaries
- `security/knowledge-owasp-top-10` — defensive programming as security control
- `security/knowledge-secure-by-design` — defense in depth architecture
