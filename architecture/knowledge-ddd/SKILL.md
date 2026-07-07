---
name: knowledge-ddd
description: >-
  Domain-Driven Design: strategic patterns (bounded contexts, context mapping,
  ubiquitous language) and tactical patterns (aggregates, entities, value
  objects, domain events, repositories). Decision framework and anti-patterns.
  Use when modelling complex domains or defining service boundaries.
---

# Domain-Driven Design (DDD)

DDD aligns software design with business domains. Apply it when the domain is complex and the model is the competitive advantage — not for simple CRUD.

## Strategic DDD

### Ubiquitous Language
- A shared vocabulary between developers and domain experts
- Every term has one precise meaning within a bounded context
- Code uses the same words as the business: class names, method names, event names
- If the team says "order" but the code says "purchase_request", the language isn't ubiquitous

### Bounded Contexts
- A boundary within which a model is consistent and the ubiquitous language applies
- Same word can mean different things in different contexts (e.g. "Account" in Billing vs Identity)
- One team typically owns one bounded context
- A microservice often maps 1:1 to a bounded context (but doesn't have to)

### Context Mapping

Relationships between bounded contexts:

| Pattern | Description |
|---------|-------------|
| **Shared Kernel** | Two contexts share a small common model (tight coupling, co-evolve) |
| **Customer-Supplier** | Upstream serves downstream; downstream can negotiate |
| **Conformist** | Downstream conforms to upstream's model (no negotiation power) |
| **Anti-Corruption Layer (ACL)** | Downstream translates upstream's model to protect its own |
| **Open Host Service** | Upstream provides a stable public API for many consumers |
| **Published Language** | Shared schema (e.g. protobuf, JSON schema) for integration |
| **Separate Ways** | No integration — contexts are independent |

**Default to ACL** when integrating with external or legacy systems.

## Tactical DDD

### Entities
- Objects with a persistent identity (ID survives state changes)
- Equality by identity, not attributes
- Example: `User(id=123)` — same user even if name changes

### Value Objects
- Objects defined entirely by their attributes (no identity)
- Immutable; equality by value
- Example: `Money(amount=100, currency="USD")`, `Address`, `EmailAddress`
- Prefer value objects over primitives for domain concepts

### Aggregates
- Cluster of entities and value objects treated as a single consistency unit
- One entity is the **aggregate root** — all external access goes through it
- Invariants are enforced within the aggregate boundary
- **Rule of thumb**: keep aggregates small; reference other aggregates by ID only

### Domain Events
- Record of something meaningful that happened in the domain
- Named in past tense: `OrderPlaced`, `PaymentReceived`, `InventoryDepleted`
- Enable decoupled communication between aggregates and bounded contexts
- Immutable once published

### Repositories
- Abstraction for aggregate persistence
- One repository per aggregate root
- Interface defined in domain layer; implementation in infrastructure
- Returns and accepts aggregate roots, not raw data

### Domain Services
- Stateless operations that don't naturally belong to an entity or value object
- Used when an operation spans multiple aggregates
- Example: `TransferService.transfer(from, to, amount)`

## When to Use DDD

| Signal | Recommendation |
|--------|---------------|
| Complex domain logic, many business rules | Full strategic + tactical DDD |
| Multiple teams, need clear ownership boundaries | Strategic DDD (bounded contexts, context maps) |
| Rich domain but single team | Tactical patterns only |
| Simple CRUD, data-centric app | Skip DDD — adds unnecessary complexity |
| Integrating legacy system | ACL pattern from strategic DDD |

## Anti-Patterns

- **Anaemic Domain Model** — entities are data bags; all logic in services. Use rich domain models instead.
- **God Aggregate** — aggregate too large, contains everything. Split along invariant boundaries.
- **Sharing aggregates across contexts** — breaks bounded context isolation. Use ACL or events.
- **CRUD-ifying DDD** — naming things `XxxService.createYyy()` when the domain has richer language (e.g. `Loan.disburse()` not `LoanService.createDisbursement()`)
- **Premature DDD** — applying full tactical patterns to a trivial domain adds cost with no benefit
- **Ignoring ubiquitous language** — developers invent their own terms instead of using domain expert vocabulary

## Cross-References

- See `architecture/knowledge-event-driven` — domain events are the bridge to event-driven architecture
- See `architecture/knowledge-clean-architecture` — entities layer maps to DDD's domain model
