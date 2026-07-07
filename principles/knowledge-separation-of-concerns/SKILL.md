---
name: knowledge-separation-of-concerns
description: >-
  Separation of concerns, cohesion, and coupling principles. Covers how to draw
  boundaries between modules, identify misplaced responsibilities, and measure
  design quality through coupling metrics. Use when decomposing systems,
  designing module boundaries, or refactoring tangled code.
---

# Separation of Concerns

## Purpose

Guide boundary decisions at every level — functions, classes, modules, services — using cohesion and coupling as measurable design qualities.

## Core Concept

A concern is a distinct aspect of functionality. Separation means each module addresses one concern without mixing unrelated logic.

## Cohesion (What Belongs Together)

High cohesion = elements in a module are strongly related and change together.

| Cohesion type | Quality | Example |
|---------------|---------|---------|
| Functional | Best | All methods serve one purpose (e.g., `OrderValidator`) |
| Sequential | Good | Output of one step feeds next (e.g., pipeline stages) |
| Communicational | Acceptable | Operate on same data (e.g., `UserProfile` read/write) |
| Temporal | Poor | Grouped by when they run (e.g., `StartupInitializer`) |
| Logical | Bad | Grouped by category (e.g., `Utils`, `Helpers`) |
| Coincidental | Worst | No meaningful relationship (e.g., `MiscService`) |

**Smell test**: If you can't name the module without using "and", "or", "misc", or "utils", cohesion is low.

## Coupling (What Should Be Independent)

Low coupling = modules can change independently.

| Coupling type | Risk | Example |
|---------------|------|---------|
| Data | Low | Passing primitive values or DTOs |
| Stamp | Low-Med | Passing a larger structure than needed |
| Control | Medium | Passing flags that alter behaviour |
| External | Medium | Shared dependency on external system |
| Common | High | Shared global/mutable state |
| Content | Highest | One module reaches into another's internals |

**Reduction strategies**:
- Depend on abstractions (interfaces) not implementations
- Use events/messages for cross-boundary communication
- Define clear public APIs; hide internals
- Pass only what's needed (data coupling over stamp coupling)

## Boundary Decision Framework

When deciding where to draw a boundary:

1. **Change frequency**: Things that change together should live together
2. **Change reason**: Things that change for different reasons should be separate
3. **Team ownership**: Boundaries often align with team boundaries (Conway's Law)
4. **Deployment independence**: Can this deploy without the other?
5. **Data ownership**: Who owns this data? That's a boundary.

## Levels of Separation

| Level | Mechanism | Example |
|-------|-----------|---------|
| Function | Extract method | Pure function for calculation |
| Class/Module | Interface | Repository interface |
| Package | Public API surface | `internal/` vs `pkg/` in Go |
| Service | Network boundary | Separate deployable |
| System | Protocol | Third-party integration |

**Principle**: Start with the lightest boundary that achieves the separation. Upgrade to stronger boundaries only when needed.

## Common Violations

- **UI logic in business layer**: Formatting, display concerns in domain code
- **Business logic in controllers**: Validation, transformation in HTTP handlers
- **Infrastructure in domain**: Database queries in business logic
- **Cross-cutting concerns scattered**: Auth checks duplicated in every handler

## Related Skills

- `architecture/knowledge-clean-architecture` — separation enforced via dependency rule
- `architecture/knowledge-ddd` — bounded contexts as separation mechanism
- `principles/knowledge-solid` — SRP is separation at the class level
