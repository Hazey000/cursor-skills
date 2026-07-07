---
name: execution-unit-testing
description: >-
  Step-by-step guide to writing effective unit tests. Covers Arrange-Act-Assert,
  naming conventions, isolation, mocking strategy, property-based testing,
  parameterized tests, and common anti-patterns. Use when writing, reviewing,
  or improving unit tests.
---

# Unit Testing

## Core Principle

**Test behaviour, not implementation.** A good unit test breaks when the code is wrong, not when the code is refactored.

## The Arrange-Act-Assert Pattern

Every unit test has exactly three phases:

```python
def test_discount_applied_for_orders_over_100():
    # Arrange — set up inputs and dependencies
    order = Order(items=[Item(price=60), Item(price=50)])
    pricer = Pricer(discount_threshold=100, discount_pct=0.1)

    # Act — execute the behaviour under test
    total = pricer.calculate_total(order)

    # Assert — verify the outcome
    assert total == 99.0  # 110 - 10% discount
```

Rules:
- **One Act per test** — if you have multiple Act steps, split into multiple tests
- **Arrange can be shared** via fixtures/setup when tests share context
- **Assert the outcome, not the journey** — don't assert intermediate state

## Naming Conventions

Use a consistent pattern that reads as a specification:

```
test_<unit>_<scenario>_<expected_result>
```

Examples:
- `test_calculate_total_with_discount_returns_reduced_price`
- `test_parse_date_with_invalid_format_raises_value_error`
- `test_user_registration_with_duplicate_email_returns_conflict`

For BDD-style frameworks: `describe/it` or `given/when/then` blocks achieve the same goal.

## Test Isolation

Each test must run independently — no shared mutable state, no ordering dependencies.

| Technique | When to use |
|-----------|-------------|
| Fresh instances in each test | Default approach |
| Setup/teardown hooks | Shared expensive setup (DB connection pool) |
| Factory functions / builders | Complex object construction |
| Fixtures (pytest, JUnit) | Reusable across test modules |

**Red flag**: if reordering tests causes failures, you have shared state.

## Mocking Strategy

### When to Mock

| Mock | Don't mock |
|------|------------|
| External services (HTTP APIs, DBs, queues) | The unit under test |
| Non-deterministic inputs (time, randomness) | Simple value objects |
| Slow dependencies (file I/O, network) | Pure functions |
| Dependencies you don't own | Dependencies you do own (prefer fakes) |

### Mock Boundaries

Mock at the **architectural boundary**, not deep inside the unit:

```
GOOD: Mock the repository interface → test the service logic
BAD:  Mock internal private methods → tests break on refactor
```

### Prefer Fakes Over Mocks

A fake is a simplified working implementation. Fakes verify behaviour naturally; mocks only verify interactions.

```python
# Fake — implements the interface with in-memory storage
class FakeUserRepo:
    def __init__(self):
        self.users = {}
    def save(self, user):
        self.users[user.id] = user
    def find(self, user_id):
        return self.users.get(user_id)

# Mock — verifies method was called (brittle)
mock_repo.save.assert_called_once_with(user)  # breaks if save is renamed
```

### Mock Rules

1. Never mock what you don't own — wrap it in an adapter, mock the adapter
2. Never mock value objects — just construct them
3. If mocking is painful, the design needs improvement (too many dependencies)
4. Verify behaviour, not call counts — `assert_called_with` > `assert_called_once`

## Parameterized Tests

When the same logic needs testing with multiple inputs, parameterize instead of duplicating:

```python
@pytest.mark.parametrize("input_val,expected", [
    ("2024-01-15", date(2024, 1, 15)),
    ("2024-12-31", date(2024, 12, 31)),
    ("2024-02-29", date(2024, 2, 29)),  # leap year
])
def test_parse_date_valid_formats(input_val, expected):
    assert parse_date(input_val) == expected
```

Use for: input validation, boundary conditions, format variations, error codes.

## Property-Based Testing

Instead of specifying examples, specify **properties** that must hold for all inputs:

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_preserves_length(xs):
    assert len(sorted(xs)) == len(xs)

@given(st.lists(st.integers()))
def test_sort_is_idempotent(xs):
    assert sorted(sorted(xs)) == sorted(xs)
```

Use for: parsers, serializers, encoders/decoders, mathematical operations, data transformations where invariants are known.

## Anti-Patterns

### Testing Implementation Details
**Symptom**: test breaks when you refactor without changing behaviour.
**Fix**: test inputs → outputs, not internal method calls or field assignments.

### Brittle Tests
**Symptom**: unrelated code changes break tests.
**Fix**: reduce coupling — mock at boundaries, assert on outcomes, use builders for test data.

### God Tests
**Symptom**: one test method with 50+ lines, multiple Act steps, dozens of assertions.
**Fix**: split into focused tests, each testing one scenario.

### Testing the Mock
**Symptom**: test passes because the mock returns what you told it to return.
**Fix**: verify the mock setup matches real behaviour. Better yet, use a fake.

### Excessive Setup
**Symptom**: 30 lines of arrange for 1 line of act.
**Fix**: builder pattern, factory functions, or object mother pattern to create test data.

### Copy-Paste Tests
**Symptom**: identical tests with minor variations.
**Fix**: parameterized tests.

## Checklist

Before merging unit tests, verify:

- [ ] Each test has a single clear reason to fail
- [ ] Test names describe the scenario and expected outcome
- [ ] No shared mutable state between tests
- [ ] Mocks are at architectural boundaries, not internal methods
- [ ] No assertions on implementation details (method call order, private state)
- [ ] Edge cases covered: nulls, empty collections, boundary values, error paths
- [ ] Parameterized tests used where 3+ tests share the same structure
- [ ] Tests run in < 5s total for the module
- [ ] No flaky tests (run suite 3x to verify)
- [ ] Test failure messages are clear enough to diagnose without reading the test code

## Cross-References

- `testing/knowledge-test-strategy` — where unit tests fit in the test pyramid
- `testing/execution-tdd-workflow` — writing tests before code
