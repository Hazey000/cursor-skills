---
name: knowledge-service-patterns
description: >-
  Backend service design patterns including Repository, Unit of Work, CQRS,
  Saga, and Service Layer. Decision framework for when each pattern applies.
  Use when designing backend services, choosing between patterns, or reviewing
  service architecture.
---

# Service Patterns

## Purpose

Decision framework for selecting and combining backend architectural patterns.

## Pattern Selection Matrix

| Pattern | Use when | Avoid when |
|---------|----------|------------|
| Repository | Need data access abstraction, testability | Simple CRUD with no business logic |
| Unit of Work | Multiple repository operations must be atomic | Single-entity operations |
| Service Layer | Business logic spans multiple entities/repos | Logic belongs in the domain entity itself |
| CQRS | Read/write models diverge significantly | Simple CRUD where reads mirror writes |
| Saga | Distributed transaction across services | Single-service operations |
| Mediator | Decouple request handling from controllers | Small codebase with few handlers |

## Repository Pattern

**Purpose**: Abstract data access behind a collection-like interface.

**When it earns its cost**:
- Multiple data sources possible (swap DB, add caching layer)
- Unit testing business logic without a database
- Complex query logic that deserves encapsulation

**When to skip**: Thin CRUD services where the ORM/query builder IS the repository.

## CQRS

**Purpose**: Separate the read model from the write model.

**Levels of CQRS**:
1. **Code-level**: Separate query and command handlers (low cost, high value)
2. **Data-level**: Separate read and write database models
3. **Infrastructure-level**: Separate read and write databases

Start at level 1. Upgrade only with evidence of need.

## Saga Pattern

**Purpose**: Manage distributed transactions via compensating actions.

**Choreography** (event-driven): Each service publishes events, others react.
- Simpler for 2-3 step flows
- Hard to understand with many steps

**Orchestration** (central coordinator): One service drives the flow.
- Clearer for complex multi-step flows
- Single point of failure for the flow

## Anti-Patterns

- **Anemic domain model**: Entities with only getters/setters, all logic in services
- **God service**: One service handling everything for a domain
- **Repository per table**: Repositories should align with aggregate roots, not tables
- **CQRS everywhere**: Most systems don't need separated read/write models

## Related Skills

- `architecture/knowledge-ddd` — aggregate roots define repository boundaries
- `architecture/knowledge-event-driven` — event-driven saga implementation
- `architecture/knowledge-clean-architecture` — where patterns sit in layers
