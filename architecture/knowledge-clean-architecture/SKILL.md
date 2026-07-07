---
name: knowledge-clean-architecture
description: >-
  Clean Architecture principles: dependency rule, layers, boundaries, and
  screaming architecture. Decision framework for when to apply full Clean
  Architecture vs simplified layering. Use when discussing service structure,
  layer separation, or dependency direction.
---

# Clean Architecture

Robert C. Martin's Clean Architecture organises code into concentric layers where **dependencies point inward** — outer layers know about inner layers, never the reverse.

## The Dependency Rule

> Source code dependencies must point inward, toward higher-level policies.

Nothing in an inner layer can reference anything in an outer layer — not a name, a type, a function, or a module. Data crossing boundaries flows via simple DTOs or interfaces defined by the inner layer.

## Layers (Inside → Outside)

### 1. Entities (Enterprise Business Rules)
- Domain objects encapsulating critical business rules
- Zero dependencies on anything external
- Survive framework changes, DB changes, UI changes

### 2. Use Cases (Application Business Rules)
- Orchestrate flow of data to/from entities
- Define application-specific rules (not business rules)
- One use case = one application action
- Depend only on entities

### 3. Interface Adapters
- Convert data between the format convenient for use cases and the format required by external agents (DB, web, etc.)
- Controllers, presenters, gateways, repository implementations
- Depend on use cases and entities

### 4. Frameworks & Drivers
- Web framework, database drivers, message brokers, UI framework
- Glue code only — minimal logic
- The most volatile layer

## Boundaries

A boundary is a line separating components with different rates of change. Enforce with:
- **Interfaces** — inner layers define ports; outer layers implement them
- **Dependency Injection** — wire implementations at composition root
- **Module/package boundaries** — enforce via build system (separate packages, internal visibility)

## Screaming Architecture

The top-level directory structure should **scream** the use cases of the system, not the framework:

```
# BAD — screams "Rails"         # GOOD — screams "Clinic Scheduling"
app/                             appointments/
  controllers/                     book_appointment.py
  models/                          cancel_appointment.py
  views/                         patients/
                                   register_patient.py
                                   update_insurance.py
```

## Decision Framework

| Signal | Recommendation |
|--------|---------------|
| Long-lived system, evolving requirements | Full Clean Architecture |
| Multiple delivery mechanisms (API, CLI, queue consumer) | Full — adapters layer pays for itself |
| Simple CRUD, stable requirements | Simplified 2-3 layer structure |
| Prototype / throwaway | Skip — add layering when validated |
| Team > 3 engineers on same service | Full — boundaries prevent coupling |
| Single delivery mechanism, tight deadline | Simplified — extract layers later |

### Simplified Layering (When Full Is Overkill)

```
handlers/       ← HTTP/gRPC handlers (thin, delegates immediately)
service/        ← business logic
repository/     ← data access (interface + impl)
```

Still follows the dependency rule, just fewer layers.

## Anti-Patterns

- **Leaking frameworks inward** — ORM entities used as domain entities
- **Anaemic use cases** — use case just calls repository.save() with no logic
- **Shared mutable models across layers** — breaks isolation
- **Circular dependencies** — always signals a boundary violation

## Cross-References

- See `architecture/knowledge-hexagonal-architecture` — ports & adapters is the same core idea, different vocabulary
- See `principles/knowledge-solid` — Dependency Inversion Principle underpins the dependency rule
