---
name: knowledge-test-strategy
description: >-
  Test strategy principles: test pyramid, coverage strategy, test ROI, and
  testing economics. Decision frameworks for test investment, when to break
  the pyramid, and how to balance testing cost against bug cost. Use when
  deciding what to test, how much to test, or structuring a test suite.
---

# Test Strategy

## The Test Pyramid

Canonical ordering by volume (most → fewest):

```
         /  E2E  \          ← few, slow, expensive, high confidence
        /----------\
       / Integration \      ← moderate count, real dependencies
      /----------------\
     /    Unit Tests     \  ← many, fast, cheap, isolated
    /____________________\
```

### What to Test at Each Level

| Level | Scope | Tests should answer | Speed target |
|-------|-------|---------------------|--------------|
| **Unit** | Single function/class, no I/O | Does this logic produce correct output? | < 5ms each |
| **Integration** | 2+ components, real I/O (DB, HTTP, queues) | Do these components work together correctly? | < 2s each |
| **E2E** | Full system through external interface | Does the user journey work end-to-end? | < 30s each |

### When to Break the Pyramid

The classic pyramid assumes most complexity lives in isolated logic. That's not always true.

**Testing Diamond** — heavy integration layer, thin unit layer. Use when:
- Business logic is mostly data transformation through external systems (ETL, API orchestration)
- The integration seams are where bugs actually live
- Unit tests would be mostly mocks testing mock behaviour

**Testing Trophy** (Kent C. Dodds) — integration-heavy with static analysis replacing some unit tests. Use when:
- Strong type system catches what unit tests would catch
- UI components where integration tests cover realistic user interaction
- Static analysis (linters, type checkers) handles the base layer

**Decision**: match test distribution to where your **actual defect risk** lives, not dogma.

## Coverage Strategy

### Critical Paths Over Line Coverage

Line coverage is a vanity metric. Optimise for **risk-weighted coverage**:

1. **Map critical paths** — revenue flows, auth, data integrity, payment processing
2. **Cover critical paths thoroughly** — happy path + known edge cases + error paths
3. **Cover everything else at the unit level** — basic correctness, not exhaustive
4. **Skip trivial code** — getters, DTOs, config objects, generated code

### Coverage Targets (Pragmatic)

| Code category | Target | Rationale |
|---------------|--------|-----------|
| Core business logic | 90%+ | Bugs here are expensive |
| API handlers / controllers | 70-80% | Integration tests cover the rest |
| Utilities / helpers | 80%+ | Reused widely, easy to unit test |
| Glue code / config | Don't measure | ROI is negative |
| Generated code | 0% | Test the generator, not the output |

## Test ROI Framework

Every test has a cost and a value. Evaluate both:

```
Test ROI = (Probability of catching a bug × Cost of that bug) / Cost of writing + maintaining the test
```

### Cost of a Test

- **Write time** — initial implementation
- **Maintenance** — updating when code changes (the dominant cost)
- **Execution time** — CI minutes, developer wait time
- **Cognitive load** — understanding what the test does when it fails

### Cost of a Bug

- **Detection stage** — caught in dev (cheap) vs production (expensive)
- **Blast radius** — one user vs all users
- **Domain** — cosmetic vs data corruption vs security breach
- **Recovery** — auto-recoverable vs manual intervention vs unrecoverable

### Investment Decision Matrix

| Scenario | Test investment | Reasoning |
|----------|----------------|-----------|
| Core payment logic | Heavy (unit + integration + e2e) | Bug cost is catastrophic |
| Internal admin tool | Light (unit + smoke test) | Low user count, easy to hotfix |
| Experimental feature behind flag | Minimal (unit only) | May be deleted next sprint |
| Shared library used by 10 services | Heavy (unit + contract) | Bug cost multiplied by consumers |
| One-off data migration | Integration test the migration, delete after | Temporary code |

## Testing Economics

### The Diminishing Returns Curve

Test value follows a logarithmic curve — the first 70% of coverage catches most bugs. Going from 90% → 95% costs disproportionately more than 50% → 70%.

**Practical implication**: invest the saved effort into better tests (more edge cases on critical paths) rather than more tests (higher line coverage on low-risk code).

### The Maintenance Tax

Every test you write is code you maintain. Tests tightly coupled to implementation details have high maintenance cost and low bug-catching value. Prefer testing behaviour (inputs → outputs) over structure.

### Flaky Test Economics

A flaky test is **worse than no test** — it erodes trust, wastes CI time, and trains developers to ignore failures. Fix or delete flaky tests immediately; never skip them indefinitely.

## Cross-References

- `testing/execution-unit-testing` — writing effective unit tests
- `testing/execution-integration-testing` — integration test design and fixtures
- `testing/execution-contract-testing` — consumer-driven contract testing
- `testing/execution-tdd-workflow` — red-green-refactor cycle
