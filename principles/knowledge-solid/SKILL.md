---
name: knowledge-solid
description: >-
  SOLID principles with practical decision frameworks for when to apply each.
  Covers Single Responsibility, Open/Closed, Liskov Substitution, Interface
  Segregation, and Dependency Inversion with real trade-offs. Use when designing
  classes, modules, or service boundaries.
---

# SOLID Principles

## Purpose

Provide decision frameworks for applying SOLID principles pragmatically — knowing when strict adherence helps and when it introduces unnecessary complexity.

## Single Responsibility Principle (SRP)

**Rule**: A module should have one, and only one, reason to change.

**Decision framework**:
- Identify the actors (stakeholders) who would request changes
- If two different actors could independently cause changes to the same module, split it
- If a class has multiple public methods that serve different use cases, consider separation

**Anti-pattern**: Splitting so aggressively that related logic scatters across dozens of tiny classes (shotgun surgery).

**Pragmatic test**: "If I change this for Actor A, could it break Actor B?"

## Open/Closed Principle (OCP)

**Rule**: Open for extension, closed for modification.

**Decision framework**:
- Apply when you've already seen a pattern of similar changes (e.g., adding new payment providers)
- Don't pre-apply speculatively — wait for the second or third instance
- Strategy pattern and plugin architectures are canonical implementations

**Anti-pattern**: Premature abstraction. Creating extension points before you have evidence they're needed.

**Pragmatic test**: "Have I changed this module for the same kind of reason more than once?"

## Liskov Substitution Principle (LSP)

**Rule**: Subtypes must be substitutable for their base types without altering correctness.

**Decision framework**:
- If your subclass throws exceptions the parent doesn't, you're violating LSP
- If your subclass ignores or no-ops parent methods, reconsider the hierarchy
- Prefer composition over inheritance when LSP is hard to maintain

**Violations to watch for**:
- Overriding methods to throw `NotImplementedException`
- Square extending Rectangle (classic example)
- Subtypes with stronger preconditions or weaker postconditions

## Interface Segregation Principle (ISP)

**Rule**: No client should be forced to depend on methods it does not use.

**Decision framework**:
- If implementors frequently leave methods empty or throw, the interface is too fat
- Split interfaces by client usage patterns, not by domain object
- Role interfaces (e.g., `Readable`, `Writable`) over header interfaces (e.g., `IFile`)

**Pragmatic test**: "Do all implementations meaningfully implement every method?"

## Dependency Inversion Principle (DIP)

**Rule**: High-level modules should not depend on low-level modules. Both should depend on abstractions.

**Decision framework**:
- Apply at architectural boundaries (domain ↔ infrastructure)
- Don't apply within a single layer for simple collaborators
- The abstraction should be owned by the high-level module (not the low-level one)

**When to skip**: Internal utilities, value objects, simple data transformations where indirection adds no testability or flexibility benefit.

## Applying SOLID Together

| Situation | Primary principle | Supporting |
|-----------|------------------|------------|
| Class doing too many things | SRP | ISP |
| Need to add a new variant without changing existing code | OCP | DIP |
| Inheritance hierarchy getting awkward | LSP | Composition |
| Fat interface with partial implementations | ISP | SRP |
| Hardcoded dependency on infrastructure | DIP | OCP |

## When SOLID is Overkill

- Scripts and CLI tools under 200 lines
- Throwaway spikes and PoCs (see `poc/knowledge-poc-methodology`)
- Internal glue code that has one consumer and changes rarely
- Performance-critical hot paths where indirection costs matter

## Related Skills

- `architecture/knowledge-clean-architecture` — SOLID at the architectural level
- `architecture/knowledge-design-patterns` — patterns that implement SOLID
- `principles/knowledge-dry-kiss-yagni` — tension between SOLID and simplicity
