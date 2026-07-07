---
name: knowledge-hexagonal-architecture
description: >-
  Hexagonal Architecture (Ports & Adapters): primary/secondary ports,
  driving/driven adapters, testability benefits, and relationship to Clean
  Architecture. Decision framework for when hexagonal is the right fit.
  Use when discussing service boundaries, adapter patterns, or testability.
---

# Hexagonal Architecture (Ports & Adapters)

Alistair Cockburn's pattern: the application is a hexagon. The inside contains business logic. The outside communicates through **ports** (interfaces) with **adapters** (implementations).

## Core Concept

```
         ┌─────────────────────────────────────┐
         │           ADAPTERS (outer)           │
         │  ┌─────────────────────────────┐    │
         │  │        PORTS (boundary)      │    │
         │  │  ┌─────────────────────┐    │    │
         │  │  │   APPLICATION CORE  │    │    │
         │  │  │   (business logic)  │    │    │
         │  │  └─────────────────────┘    │    │
         │  └─────────────────────────────┘    │
         └─────────────────────────────────────┘
```

The application defines ports. External concerns plug in as adapters. The core never knows which adapter is connected.

## Ports

A port is an **interface** defined by the application core. It describes *what* the application needs, not *how* it's fulfilled.

### Primary Ports (Driving)
- Define how the **outside world drives the application**
- Typically: use case interfaces, command handlers, API contracts
- Implemented by the application core itself
- Called by driving adapters (HTTP controllers, CLI, test harness)

### Secondary Ports (Driven)
- Define what the **application needs from the outside**
- Typically: repository interfaces, notification sender, payment gateway
- Defined by the core, implemented by adapters
- Called by the application core

## Adapters

### Driving Adapters (Primary/Left-side)
- Translate external input into calls on primary ports
- Examples: REST controller, GraphQL resolver, message consumer, CLI parser
- Contain no business logic — transform and delegate

### Driven Adapters (Secondary/Right-side)
- Implement secondary ports
- Examples: PostgreSQL repository, Redis cache adapter, SMTP email sender, S3 file store
- Swappable without touching core logic

## Testability

The primary benefit: **test the core in isolation**.

| Test type | Approach |
|-----------|----------|
| Unit tests (core logic) | Call primary ports directly, stub secondary ports with in-memory fakes |
| Integration tests | Use real driven adapters against test infrastructure |
| Acceptance tests | Drive via primary adapters (HTTP client), real or faked driven adapters |

Because ports are interfaces, fakes/mocks are trivial to create.

## Mapping to Clean Architecture

| Hexagonal | Clean Architecture |
|-----------|-------------------|
| Application core | Entities + Use Cases |
| Primary ports | Use Case input boundaries |
| Secondary ports | Gateway/Repository interfaces |
| Driving adapters | Controllers (Interface Adapters) |
| Driven adapters | Gateways, Repos (Interface Adapters / Frameworks) |

Same dependency rule. Hexagonal emphasises symmetry (all sides equal); Clean Architecture emphasises concentric layers.

## Typical Project Structure

```
cmd/                        ← entry points (wires adapters)
internal/
  domain/                   ← entities, value objects (pure)
  application/              ← use cases, port interfaces
    ports/
      driving.go            ← primary port interfaces
      driven.go             ← secondary port interfaces
  adapters/
    driving/
      http/                 ← REST adapter
      grpc/                 ← gRPC adapter
    driven/
      postgres/             ← DB adapter
      redis/                ← cache adapter
      kafka/                ← event publisher adapter
```

## Decision Framework

| Signal | Recommendation |
|--------|---------------|
| Multiple inbound channels (HTTP + queue + CLI) | Hexagonal — driving adapters share the same core |
| Need to swap infrastructure (migrate DB, change cloud provider) | Hexagonal — swap adapters without touching core |
| Heavy testing requirements, CI speed matters | Hexagonal — in-memory adapters make tests fast |
| Simple single-entry microservice, stable infra | May be overkill — simpler layering suffices |
| Team unfamiliar with pattern | Start with simplified version (ports only for driven side) |

## Common Mistakes

- **Fat adapters** — business logic creeps into adapters; keep them as thin translators
- **Port explosion** — one interface per method; group related operations into cohesive ports
- **Shared domain objects in API contracts** — never expose domain types directly through driving adapters
- **Skipping the port** — calling infrastructure directly from use cases defeats the purpose

## Cross-References

- See `architecture/knowledge-clean-architecture` — same dependency direction, layered vocabulary
- See `architecture/knowledge-ddd` — domain layer maps to the hexagonal core
