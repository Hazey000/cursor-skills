---
name: execution-service-design
description: >-
  Step-by-step procedure to design a new service from requirements. Covers
  boundary identification, interface design, architecture style selection,
  data modelling, error handling, observability, and testing strategy.
  Use when starting a new service, microservice, or significant module.
---

# Service Design Procedure

A repeatable procedure for designing a new service from requirements through to implementation readiness.

## Procedure

### Step 1: Identify Boundaries

Determine what this service is responsible for — and what it is NOT.

- [ ] Name the bounded context (one noun phrase: "Payment Processing", "User Identity")
- [ ] List the service's **responsibilities** (what it owns)
- [ ] List explicit **non-responsibilities** (what belongs elsewhere)
- [ ] Identify upstream dependencies (services this one calls)
- [ ] Identify downstream consumers (services that call or subscribe to this one)
- [ ] Define the aggregate root(s) this service manages

**Heuristic**: if two capabilities change for different reasons or at different rates, they belong in different services.

### Step 2: Define Interfaces

Specify how the outside world interacts with this service.

- [ ] **Inbound APIs**: REST, gRPC, GraphQL, message consumer — pick based on consumer needs
- [ ] **Outbound events**: what domain events does this service publish?
- [ ] **Outbound calls**: what other services/APIs does it call?
- [ ] Define API contracts (request/response schemas, error codes)
- [ ] Version strategy (URL path, header, protobuf field numbering)

**Heuristic**: design the interface from the consumer's perspective, not the implementation's.

### Step 3: Choose Architecture Style

Select the internal architecture based on complexity.

| Complexity | Recommended style |
|------------|------------------|
| Simple CRUD, < 5 entities | Layered (handler → service → repo) |
| Moderate domain logic, multiple entry points | Hexagonal / Ports & Adapters |
| Complex domain, many business rules | Clean Architecture + DDD tactical patterns |
| Event-heavy, audit-required | Event Sourcing + CQRS |

- [ ] Document chosen style and rationale
- [ ] Define the package/module structure upfront

### Step 4: Design Data Model

- [ ] Identify entities and value objects
- [ ] Choose primary data store (relational, document, key-value, event store)
- [ ] Design schema / collection structure
- [ ] Identify indices needed for query patterns
- [ ] Plan data access patterns (read-heavy? write-heavy? both?)
- [ ] Consider caching strategy (none, read-through, write-behind)
- [ ] Plan for data migrations (schema versioning approach)

**Heuristic**: choose the store that matches your access pattern, not the one you're most familiar with.

### Step 5: Plan Error Handling

- [ ] Define error categories: validation, business rule, infrastructure, upstream failure
- [ ] Map errors to appropriate response codes (4xx vs 5xx for HTTP; gRPC status codes)
- [ ] Define retry policy for outbound calls (exponential backoff, max retries, circuit breaker)
- [ ] Design dead letter queue / failure handling for async operations
- [ ] Define compensation/rollback strategy for distributed operations
- [ ] Plan graceful degradation (what works when a dependency is down?)

### Step 6: Define Observability

- [ ] **Metrics**: request rate, error rate, latency (p50/p95/p99), saturation (queue depth, connection pool)
- [ ] **Structured logging**: request IDs, correlation IDs, key business events
- [ ] **Distributed tracing**: propagate trace context through all inbound/outbound calls
- [ ] **Health checks**: liveness (is the process running?) and readiness (can it serve traffic?)
- [ ] **Alerts**: define SLOs and alert when error budget burns too fast
- [ ] **Dashboards**: one per service showing golden signals (rate, errors, latency, saturation)

### Step 7: Plan Testing Strategy

| Layer | What to test | Approach |
|-------|-------------|----------|
| Domain logic | Business rules, aggregates | Unit tests (no I/O, fast) |
| Use cases | Orchestration, edge cases | Unit tests with faked ports |
| Adapters | Serialisation, API contracts | Integration tests (real infra or testcontainers) |
| End-to-end | Critical paths | Contract tests + a few E2E smoke tests |

- [ ] Define test boundaries (what's mocked, what's real)
- [ ] Plan contract tests for API consumers
- [ ] Identify critical paths that need E2E coverage
- [ ] Set coverage expectations (domain layer: high; adapter layer: integration-tested)

## Design Review Checklist

Before implementation, verify:

- [ ] Service has a clear, single responsibility
- [ ] API contracts are defined and versioned
- [ ] Data model supports all known query patterns
- [ ] Error handling covers all failure modes (self, upstream, downstream)
- [ ] Observability covers golden signals
- [ ] No hard dependencies on other services' internals
- [ ] Security: AuthN/AuthZ defined for all endpoints
- [ ] Performance: known throughput requirements and plan to meet them
- [ ] Deployment: can be deployed independently of other services

## Output Artefact

Produce a **Service Design Document** covering:

1. Service name and bounded context
2. Responsibility statement (owns / does not own)
3. Architecture style and rationale
4. Interface definitions (API specs, event schemas)
5. Data model diagram
6. Dependency map (upstream/downstream)
7. Error handling strategy
8. Observability plan
9. Testing strategy
10. Open questions / risks

## Cross-References

- See `architecture/knowledge-clean-architecture` — for layered/clean style selection
- See `architecture/knowledge-hexagonal-architecture` — for ports & adapters style
- See `architecture/knowledge-ddd` — for aggregate/bounded context identification
- See `architecture/knowledge-event-driven` — for async communication and event sourcing decisions
- See `architecture/knowledge-design-patterns` — for pattern selection within the service
