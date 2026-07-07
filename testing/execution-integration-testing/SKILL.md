---
name: execution-integration-testing
description: >-
  Design and execute integration test suites. Covers scope definition, test
  containers, database testing, API testing, message queue testing, test data
  management, flakiness prevention, and speed optimization. Use when writing
  or reviewing integration tests.
---

# Integration Testing

## Core Principle

**Test the seams.** Integration tests verify that components work together correctly across real boundaries — the exact places unit tests can't reach.

## Scope Definition

An integration test exercises **2+ components interacting through a real boundary**:

| Boundary type | Example |
|---------------|---------|
| Database | Service → real DB (not mocked repo) |
| HTTP | Client → real HTTP server → handler |
| Message queue | Producer → real broker → consumer |
| File system | Service → real file I/O |
| External service | Service → real (or contract-verified) dependency |

**Not integration tests**: two classes in the same module collaborating through method calls — that's a unit test with wider scope.

## Workflow

### Step 1: Identify Integration Points

Map the boundaries in the system under test. Each boundary that carries risk warrants at least one integration test covering:
- Happy path
- Primary error/failure mode
- Edge case most likely to cause production issues

### Step 2: Set Up Infrastructure

Use containers or in-memory replacements for real dependencies.

#### Test Containers

Preferred approach — spin up real dependencies in Docker:

```python
# Python (testcontainers)
from testcontainers.postgres import PostgresContainer

@pytest.fixture(scope="session")
def db():
    with PostgresContainer("postgres:16") as pg:
        engine = create_engine(pg.get_connection_url())
        run_migrations(engine)
        yield engine
```

```java
// Java (Testcontainers)
@Container
static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:16");
```

**Key decisions**:
- `scope="session"` for containers (expensive to start) — share across tests
- `scope="function"` for data isolation — each test gets clean state
- Use health checks / wait strategies so tests don't race container startup

#### In-Memory Alternatives

| Dependency | Container | In-memory alternative |
|------------|-----------|----------------------|
| PostgreSQL | `postgres:16` | SQLite (if dialect-compatible) |
| Redis | `redis:7` | fakeredis |
| S3 | MinIO container | moto (AWS mock) |
| Kafka | `confluentinc/cp-kafka` | — (use container) |
| Elasticsearch | `elasticsearch:8` | — (use container) |

Prefer containers over in-memory fakes when behaviour differences matter (SQL dialects, transaction semantics).

### Step 3: Manage Test Data

#### Per-Test Isolation Strategies

| Strategy | How it works | Best for |
|----------|-------------|----------|
| **Transaction rollback** | Wrap each test in a transaction, rollback after | Fast, simple; breaks if code under test manages its own transactions |
| **Truncate after** | Delete all rows after each test | Works with any transaction model |
| **Unique prefixes** | Each test uses unique IDs/names to avoid collision | Parallel-safe, no cleanup needed |
| **Fresh schema per test** | Create/drop schema per test | Strongest isolation, slowest |

**Recommended default**: transaction rollback for speed, fall back to truncate when rollback isn't viable.

#### Test Data Builders

Don't hardcode test data. Use builders that create valid defaults and let tests override only what matters:

```python
def make_user(**overrides):
    defaults = {
        "id": uuid4(),
        "name": "Test User",
        "email": f"{uuid4().hex[:8]}@test.com",
        "status": "active",
    }
    return User(**(defaults | overrides))
```

### Step 4: Write the Test

Structure mirrors unit tests (Arrange-Act-Assert) but with real I/O:

```python
def test_create_order_persists_to_database(db_session):
    # Arrange
    user = make_user()
    db_session.add(user)
    db_session.flush()

    # Act
    order = order_service.create_order(user_id=user.id, items=[...])

    # Assert
    persisted = db_session.get(Order, order.id)
    assert persisted is not None
    assert persisted.user_id == user.id
    assert persisted.status == "pending"
```

### Step 5: API Testing

Test through real HTTP — don't call handler functions directly:

```python
def test_create_user_returns_201(client):
    response = client.post("/users", json={"name": "Alice", "email": "alice@test.com"})

    assert response.status_code == 201
    assert response.json()["name"] == "Alice"

def test_create_user_duplicate_email_returns_409(client, existing_user):
    response = client.post("/users", json={"name": "Bob", "email": existing_user.email})

    assert response.status_code == 409
```

Use the framework's test client (FastAPI `TestClient`, Spring `MockMvc`, Go `httptest`) — it runs a real HTTP stack without a network socket.

### Step 6: Message Queue Testing

Pattern: publish → wait for consumption → assert side effect.

```python
def test_order_event_triggers_inventory_update(broker, db_session):
    order = make_order(items=[Item(sku="ABC", qty=2)])

    broker.publish("orders.created", order.to_event())
    wait_for(lambda: db_session.get(InventoryReservation, order.id), timeout=5)

    reservation = db_session.get(InventoryReservation, order.id)
    assert reservation.sku == "ABC"
    assert reservation.quantity == 2
```

**Critical**: always use a timeout with a clear error message. Never `sleep()` for a fixed duration.

## Flakiness Prevention

| Cause | Prevention |
|-------|-----------|
| Timing / race conditions | Use polling with timeout, never `sleep()` |
| Port conflicts | Use dynamic port allocation |
| Shared state between tests | Per-test isolation (see Step 3) |
| Container startup races | Use health-check wait strategies |
| Non-deterministic ordering | Don't depend on DB row order without `ORDER BY` |
| External service flakiness | Use containers or contract stubs, not live services |
| Time-dependent logic | Inject a clock, freeze time in tests |

## Speed Optimization

| Technique | Impact |
|-----------|--------|
| Share containers across tests (session scope) | High — avoids repeated startup |
| Transaction rollback instead of truncate | Medium — faster cleanup |
| Parallel test execution by module | High — but requires full data isolation |
| Reuse DB connections via pool | Medium — reduces connection overhead |
| Run only affected integration tests in CI (test impact analysis) | High — skips unrelated tests |
| Keep integration tests in a separate directory/marker | Allows running unit tests alone for fast feedback |

## Checklist

Before merging integration tests, verify:

- [ ] Test exercises a real boundary (DB, HTTP, queue), not just in-process collaboration
- [ ] Infrastructure managed via containers or reliable in-memory alternatives
- [ ] Each test has its own isolated data — no cross-test dependencies
- [ ] No `sleep()` calls — uses polling/wait with timeout instead
- [ ] Timeouts have clear error messages explaining what was expected
- [ ] Test data uses builders with unique identifiers
- [ ] Container health checks prevent startup race conditions
- [ ] Tests are idempotent — running twice produces the same result
- [ ] Cleanup happens even when tests fail (use fixtures/hooks, not manual cleanup)
- [ ] Suite completes in < 2 minutes locally

## Cross-References

- `testing/knowledge-test-strategy` — where integration tests fit in the pyramid
- `testing/execution-contract-testing` — when contract tests can replace integration tests
- `testing/execution-unit-testing` — what should be tested at the unit level instead
