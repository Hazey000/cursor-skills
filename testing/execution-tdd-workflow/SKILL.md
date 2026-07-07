---
name: execution-tdd-workflow
description: >-
  Red-green-refactor TDD workflow with practical guidance. Covers when TDD
  helps most, when it's counterproductive, outside-in vs inside-out approaches,
  and the Transformation Priority Premise. Use when practising TDD or deciding
  whether TDD is appropriate for a task.
---

# TDD Workflow

## Core Principle

**Write the test that forces you to write the code.** TDD is a design tool that happens to produce tests — the tests drive the interface and structure of the code, not just verify it after the fact.

## The Red-Green-Refactor Cycle

```
┌─────────┐     ┌─────────┐     ┌──────────┐
│  RED     │────▶│  GREEN  │────▶│ REFACTOR │──┐
│ Write a  │     │ Make it │     │ Clean up │  │
│ failing  │     │ pass    │     │ the code │  │
│ test     │     │ (minimal│     │ (no new  │  │
│          │     │  code)  │     │  behaviour)│ │
└─────────┘     └─────────┘     └──────────┘  │
     ▲                                         │
     └─────────────────────────────────────────┘
```

### RED — Write a Failing Test

1. Write a test for the **next smallest increment** of behaviour
2. Run it — confirm it **fails for the right reason** (not a syntax error)
3. The test name should describe the behaviour you're about to implement

### GREEN — Make It Pass

1. Write the **simplest code** that makes the test pass
2. It's okay if the code is ugly — resist the urge to design at this step
3. Don't write more code than the test demands
4. Run all tests — confirm the new test passes and nothing else broke

### REFACTOR — Clean Up

1. Improve structure without changing behaviour: extract methods, rename, remove duplication
2. Run all tests after each refactoring step — they must stay green
3. This is where design emerges: patterns, abstractions, clean interfaces

**Discipline**: never skip refactor. The cycle degenerates into "write test, write code" without it.

## When TDD Helps Most

| Scenario | Why TDD works well |
|----------|-------------------|
| Complex business logic | Tests document edge cases, force you to handle them |
| Bug reproduction | Write the failing test first, then fix — proves the fix works |
| Clear input/output interfaces | Easy to express as assertions |
| Algorithm design | Small steps prevent getting lost in complexity |
| Shared libraries / APIs | Tests drive the consumer-facing interface |
| Unfamiliar domain | Tests are a safe way to explore and learn incrementally |

## When TDD Is Counterproductive

| Scenario | Why | Alternative |
|----------|-----|-------------|
| Exploratory / spike code | You don't know what the interface should be yet | Spike first, then write tests for the keeper code |
| UI layout / styling | Visual correctness isn't expressible as unit assertions | Visual regression tests, snapshot tests |
| Integration glue code | The logic is in the wiring, not the components | Integration tests after the fact |
| Prototyping throwaway code | Tests for code you'll delete tomorrow | Skip tests entirely |
| Framework boilerplate | Generated or formulaic code with no logic | Coverage via integration/e2e tests |

**Rule of thumb**: if you can't describe the expected behaviour in a test name, TDD is premature. Spike until you can.

## Transformation Priority Premise

When going from RED to GREEN, apply transformations in order of increasing complexity. This guides you toward the simplest implementation at each step:

| Priority | Transformation | Example |
|----------|---------------|---------|
| 1 | `{}` → nil/constant | Return a hardcoded value |
| 2 | Constant → scalar variable | Replace constant with a parameter |
| 3 | Statement → statements | Add unconditional logic |
| 4 | Unconditional → conditional | Add an `if` |
| 5 | Scalar → collection | Single value → list/array |
| 6 | Statement → recursion/iteration | Add a loop |
| 7 | Value → mutation | Accumulate state |

**Key insight**: if you're jumping to priority 6 (loops) when the tests only demand priority 2 (variables), you're writing more code than needed. Add tests that incrementally force higher-priority transformations.

## Outside-In vs Inside-Out TDD

### Outside-In (London School / Mockist)

Start from the outer layer (controller/handler) and work inward. Mock dependencies that don't exist yet.

```
1. Write acceptance test for the feature (failing)
2. Write controller test — mock the service
3. Implement controller, discover service interface
4. Write service test — mock the repository
5. Implement service, discover repository interface
6. Write repository test against real DB
7. Implement repository
8. Acceptance test passes
```

**Best for**: systems with clear layers, API-first design, when you want the outer interface to drive inner structure.

**Risk**: over-mocking, tests coupled to interaction patterns.

### Inside-Out (Detroit School / Classicist)

Start from the core logic and build outward. No mocks — use real collaborators.

```
1. Write test for core domain logic (no dependencies)
2. Implement domain logic
3. Write test for service that uses domain logic
4. Implement service (uses real domain objects)
5. Write test for controller that uses service
6. Implement controller (uses real service)
```

**Best for**: domain-heavy systems, when core logic is the hard part, when you want tests decoupled from structure.

**Risk**: may discover late that the outer interface doesn't fit.

### Choosing

| Factor | Outside-In | Inside-Out |
|--------|-----------|------------|
| Where is the complexity? | In the wiring / coordination | In the domain logic |
| Do you know the external interface? | Yes (API spec, UI mockup) | Not yet |
| Team mocking comfort | Comfortable with mocks | Prefers real collaborators |
| Integration test coverage | Need contract/integration tests too | Unit tests are often sufficient |

## Worked Example Structure

When applying TDD, document the progression:

```
## Feature: Calculate shipping cost

### Test 1: Free shipping for orders over $100
RED:   test_shipping_free_for_orders_over_100 → FAIL (no function exists)
GREEN: return 0.0 (hardcoded — only case so far)

### Test 2: Flat rate $5 for orders under $100
RED:   test_shipping_flat_rate_for_small_orders → FAIL (returns 0.0)
GREEN: if order.total > 100: return 0.0 else: return 5.0

### Test 3: Express shipping adds $10
RED:   test_express_shipping_adds_surcharge → FAIL (no shipping_method param)
GREEN: Add shipping_method parameter, add $10 for express

### Refactor
- Extract shipping rules into a strategy pattern
- All 3 tests still pass
```

Each step adds one new behaviour. The code evolves incrementally from simple to sophisticated.

## Common Mistakes

- **Too-big first test** — start with the simplest possible case, not the full happy path
- **Skipping the RED step** — if the test passes immediately, it's testing nothing new (or you wrote the code first)
- **Gold-plating in GREEN** — writing elegant code before you have enough tests to justify the structure
- **Never refactoring** — accumulating "just make it pass" code without cleanup
- **Testing implementation** — asserting on mock call counts instead of observable outcomes

## Cross-References

- `testing/knowledge-test-strategy` — test investment decisions
- `testing/execution-unit-testing` — detailed unit testing patterns used within TDD
