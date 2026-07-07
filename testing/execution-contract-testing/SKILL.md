---
name: execution-contract-testing
description: >-
  Consumer-driven contract testing workflow. Covers defining consumer
  expectations, generating contracts, verifying providers, the Pact pattern,
  and handling breaking changes. Use when testing service boundaries, API
  compatibility, or replacing heavyweight integration tests between services.
---

# Contract Testing

## Core Principle

**Test the agreement, not the implementation.** A contract test verifies that a provider and consumer agree on the shape, semantics, and constraints of their shared interface — without requiring both to run simultaneously.

## When to Use Contract Tests

| Scenario | Contract test? | Rationale |
|----------|---------------|-----------|
| Service A calls Service B's API | Yes | Catches breaking changes before deployment |
| Frontend consumes backend API | Yes | Frontend and backend release independently |
| Shared library exposes public API | Maybe | Semver + unit tests often suffice |
| Two modules in the same repo | No | Integration tests are simpler and faster |
| Third-party API you don't control | No (use a stub) | You can't verify the provider |

**Key insight**: contract tests are most valuable when **consumer and provider deploy independently** and integration tests are too expensive to run on every change.

## Consumer-Driven Contract Workflow

### Step 1: Define Consumer Expectations

The consumer writes tests describing what it **expects** from the provider — the requests it sends and the responses it requires.

```python
# Consumer test (using Pact)
def test_get_user_contract(pact):
    expected = {"id": 1, "name": "Alice", "email": "alice@example.com"}

    (pact
        .given("user 1 exists")
        .upon_receiving("a request for user 1")
        .with_request("GET", "/users/1")
        .will_respond_with(200, body=Like(expected)))

    # Run consumer code against the Pact mock server
    with pact:
        user = user_client.get_user(1)
        assert user.name == "Alice"
```

**Rules for consumer expectations**:
- Specify only the fields you **actually use** — don't assert the entire response
- Use matchers (`Like`, `EachLike`, `Term`) instead of exact values for flexibility
- Each interaction should be a single request-response pair

### Step 2: Generate the Contract

Running consumer tests produces a contract file (Pact JSON, OpenAPI spec, or Protobuf descriptor):

```json
{
  "consumer": { "name": "order-service" },
  "provider": { "name": "user-service" },
  "interactions": [
    {
      "description": "a request for user 1",
      "providerState": "user 1 exists",
      "request": { "method": "GET", "path": "/users/1" },
      "response": {
        "status": 200,
        "body": { "id": 1, "name": "Alice", "email": "alice@example.com" }
      }
    }
  ]
}
```

Publish the contract to a **broker** (Pact Broker, PactFlow) so the provider can retrieve it.

### Step 3: Verify Provider Against Contract

The provider runs a verification test that replays every interaction from the contract against its real implementation:

```python
# Provider verification
def test_provider_honours_contracts():
    verifier = Verifier(
        provider="user-service",
        provider_base_url="http://localhost:8080",
    )
    verifier.verify_with_broker(
        broker_url="https://pact-broker.internal",
        provider_states_setup_url="http://localhost:8080/_pact/setup",
    )
```

**Provider states**: the provider must implement a setup endpoint or hook that puts the system into the required state (e.g., "user 1 exists" → insert a user with id=1).

### Step 4: Integrate Into CI

```
Consumer CI:
  1. Run consumer contract tests → generate contract
  2. Publish contract to broker
  3. Run `can-i-deploy` check before deploying

Provider CI:
  1. Fetch latest contracts from broker
  2. Run verification against contracts
  3. Publish verification results
  4. Run `can-i-deploy` check before deploying
```

The `can-i-deploy` check queries the broker to confirm all contracts between this version and its dependents are verified.

## When Contracts Replace Integration Tests

Contract tests **can replace** cross-service integration tests when:
- The contract covers the exact interactions the integration test exercises
- Provider verification runs against real handler code (not a stub)
- Provider states set up realistic data

Contract tests **cannot replace** integration tests when:
- You need to verify end-to-end data flow across 3+ services
- The interaction involves complex stateful sequences (multi-step workflows)
- You're testing non-functional concerns (latency, concurrency, resource limits)

## Breaking Contract Change Workflow

When a provider needs to make a breaking change:

1. **Provider**: add the new endpoint/field alongside the old one (non-breaking)
2. **Provider**: publish new API version, deploy
3. **Consumers**: migrate to new endpoint/field, update consumer contracts
4. **Consumers**: publish updated contracts, deploy
5. **Provider**: verify no consumers reference the old endpoint/field (broker shows no active contracts)
6. **Provider**: remove deprecated endpoint/field, deploy

This is the **expand-contract** (parallel change) pattern. Never remove before all consumers migrate.

See `api/knowledge-api-versioning` for versioning strategies that support this workflow.

## Pact-Specific Guidance

### Matchers

| Matcher | Use for | Example |
|---------|---------|---------|
| `Like(value)` | Type matching (any value of same type) | `Like(42)` matches any integer |
| `EachLike(value)` | Non-empty array of matching type | `EachLike({"id": Like(1)})` |
| `Term(regex, example)` | Regex-constrained string | `Term(r"\d{4}-\d{2}-\d{2}", "2024-01-15")` |

### Provider States

Keep provider states:
- **Descriptive**: `"user 1 exists with premium subscription"` not `"state 1"`
- **Independent**: each state sets up from scratch, no ordering dependency
- **Minimal**: only create data the interaction needs

### Common Pitfalls

- **Over-specifying contracts** — asserting fields you don't use; breaks when provider adds optional fields
- **Under-specifying contracts** — not matching on types/structure; contract passes but consumer breaks at runtime
- **Skipping provider states** — provider verification against empty DB; everything returns 404
- **Not publishing to broker** — contracts stored locally; provider never verifies them

## Checklist

Before merging contract tests, verify:

- [ ] Consumer tests specify only the fields and status codes the consumer actually uses
- [ ] Matchers used instead of exact values where flexibility is needed
- [ ] Provider states are descriptive and set up realistic data
- [ ] Contract published to broker (not just stored locally)
- [ ] Provider verification runs against real handler code
- [ ] `can-i-deploy` check integrated into both consumer and provider CI
- [ ] Breaking changes follow expand-contract pattern
- [ ] No duplicate coverage — if a contract test covers an interaction, remove the equivalent integration test

## Cross-References

- `testing/knowledge-test-strategy` — where contract tests fit in the testing strategy
- `testing/execution-integration-testing` — when integration tests are still needed
- `api/knowledge-api-versioning` — versioning strategies for non-breaking evolution
