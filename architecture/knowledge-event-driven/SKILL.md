---
name: knowledge-event-driven
description: >-
  Event-driven architecture: event sourcing, CQRS, messaging patterns (pub/sub,
  event bus, message queue), eventual consistency, sagas, choreography vs
  orchestration. Decision framework for sync vs async communication.
  Use when designing async systems, choosing communication patterns, or
  discussing event sourcing.
---

# Event-Driven Architecture

Systems communicate through events — records of things that happened. Components react to events rather than being directly called.

## Core Concepts

### Event
An immutable record of a state change: `OrderPlaced { orderId, customerId, items, timestamp }`. Named in past tense. Contains enough data for consumers to act without calling back to the producer.

### Event Producer / Consumer
- **Producer**: publishes events when its state changes
- **Consumer**: subscribes to events and reacts
- Producers don't know or care who consumes their events (decoupled)

## Messaging Patterns

### Publish/Subscribe (Pub/Sub)
- Producer publishes to a topic; multiple subscribers receive a copy
- Fan-out: one event triggers multiple independent reactions
- Use for: notifications, cross-service synchronisation, analytics pipelines

### Message Queue (Point-to-Point)
- Producer sends to a queue; exactly one consumer processes each message
- Use for: work distribution, task processing, load levelling
- Guarantees: at-least-once delivery (with most brokers)

### Event Bus
- In-process pub/sub for decoupling modules within a single service
- Lightweight; no external broker needed
- Use for: intra-service decoupling in monoliths or modular monoliths

## Event Sourcing

Instead of storing current state, store the **sequence of events** that led to it. Current state is derived by replaying events.

**Benefits:**
- Complete audit trail
- Temporal queries ("what was the state at time T?")
- Rebuild read models from scratch
- Natural fit for event-driven systems

**Costs:**
- Schema evolution is hard (events are immutable)
- Replay performance degrades with long event streams (use snapshots)
- Eventual consistency everywhere
- Steep learning curve

**Use when:** audit is mandatory, temporal queries needed, or domain is naturally event-based (finance, logistics).
**Avoid when:** simple CRUD, team inexperienced, or strong consistency required.

## CQRS (Command Query Responsibility Segregation)

Separate the write model (commands) from the read model (queries). They can use different data stores, schemas, and scaling strategies.

```
Commands → Write Model (normalised, enforces invariants)
                 ↓ (events)
           Read Model (denormalised, optimised for queries)
```

**Use with event sourcing**: events from the write side project into read-optimised views.
**Use without event sourcing**: write to primary DB, project changes to read replicas/caches.

**Benefits:** independent scaling, query-optimised views, simpler write model.
**Costs:** eventual consistency between read/write, added infrastructure, data sync complexity.

## Sagas: Managing Distributed Transactions

When a business process spans multiple services, use a saga to coordinate.

### Choreography
- Each service reacts to events and emits its own events
- No central coordinator; the process emerges from event reactions
- **Pros**: simple, decoupled, no single point of failure
- **Cons**: hard to trace flow, difficult to handle failures in long chains

### Orchestration
- A dedicated orchestrator service directs the saga steps
- Sends commands to participants, handles responses
- **Pros**: explicit flow, easier error handling and compensation
- **Cons**: orchestrator is a single point of failure, tighter coupling

| Signal | Recommendation |
|--------|---------------|
| 2-3 services, simple flow | Choreography |
| 4+ services, complex branching | Orchestration |
| Need visibility into process state | Orchestration |
| Want maximum decoupling | Choreography |

### Compensation
Sagas use **compensating actions** instead of rollback. If step 3 fails, execute compensating actions for steps 2 and 1 (e.g. `RefundPayment`, `UnreserveInventory`).

## Eventual Consistency

In event-driven systems, data across services will be **eventually consistent** — not immediately. Design for this:

- UI: show optimistic updates with "processing" states
- APIs: return 202 Accepted for async operations, provide status endpoints
- Idempotency: consumers must handle duplicate events safely (use event IDs)
- Ordering: within a partition/aggregate, events are ordered; across partitions, assume unordered

## Decision Framework: Sync vs Async

| Signal | Sync (request/response) | Async (events/messages) |
|--------|------------------------|------------------------|
| Caller needs immediate answer | Yes | No |
| Operation can fail independently | No | Yes |
| Fan-out to multiple consumers | Awkward | Natural |
| Need guaranteed delivery | Already have it | Need broker with durability |
| Latency-sensitive response | Yes | No (adds latency) |
| Workload is spiky/bursty | Risk overload | Queue absorbs spikes |
| Simple request/response semantics | Yes | Overkill |

**Default**: start synchronous. Introduce async when you need decoupling, resilience, or fan-out.

## Anti-Patterns

- **Event soup** — publishing events for everything with no clear consumer; creates noise and maintenance burden
- **Distributed monolith** — services tightly coupled via event schemas; a schema change cascades everywhere
- **Event as command** — naming events as instructions (`SendEmail`) rather than facts (`OrderConfirmed`). Events describe what happened; commands describe what to do.
- **Missing idempotency** — consumers break on duplicate delivery
- **Synchronous event handling** — blocking the publisher waiting for consumers defeats the purpose

## Cross-References

- See `architecture/knowledge-ddd` — domain events are the bridge between DDD and event-driven architecture
- See `architecture/execution-service-design` — event-driven is one of the architecture styles to choose during service design
