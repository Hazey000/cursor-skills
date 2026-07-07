---
name: review-architecture
description: >-
  Evaluate architectural decisions, layer boundaries, dependency direction,
  coupling, cohesion, and scalability patterns. Produces a structured review
  report with severity-rated findings.
---

# Architecture Review

Evaluate code and system design against sound architectural principles. Focus on structural issues that compound over time.

**Cross-reference**: `architecture/knowledge-clean-architecture` for dependency rule and layer definitions.

## When to Use

- Before merging significant structural changes
- When adding a new service, module, or major feature
- When refactoring or splitting a monolith
- During design review of a proposed architecture

## Severity Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Architectural violation that will cause systemic problems — fix before merge |
| **WARNING** | Design smell that increases maintenance cost — fix soon |
| **SUGGESTION** | Improvement opportunity — consider for next iteration |

## Review Checklist

### 1. Dependency Direction

| Check | Severity if violated |
|-------|---------------------|
| Dependencies point inward (domain has no external deps) | CRITICAL |
| No circular dependencies between modules/packages | CRITICAL |
| Infrastructure depends on domain, never the reverse | CRITICAL |
| Interface adapters don't leak into business logic | WARNING |
| Framework code is isolated to outer layers | WARNING |

**Common violations**:
- Domain entity importing an ORM decorator or DB driver
- Business logic importing HTTP request/response types
- Shared `utils` package creating hidden coupling between unrelated modules

### 2. Separation of Concerns

| Check | Severity if violated |
|-------|---------------------|
| Each module/service has a single, clear responsibility | WARNING |
| Business rules are not embedded in controllers or handlers | CRITICAL |
| Data transformation is separate from data access | WARNING |
| Cross-cutting concerns (logging, auth, metrics) use middleware/decorators, not inline code | SUGGESTION |
| Configuration is separate from logic | WARNING |

**Common violations**:
- Controller that validates, queries DB, applies business rules, and formats response
- Service class exceeding ~300 lines (god service smell)
- Mixing orchestration logic with domain logic

### 3. Coupling and Cohesion

| Check | Severity if violated |
|-------|---------------------|
| Modules communicate through well-defined interfaces/contracts | WARNING |
| Changes to one module don't cascade to unrelated modules | CRITICAL |
| Related functionality is co-located (high cohesion) | WARNING |
| Shared mutable state between modules is absent | CRITICAL |
| Data structures passed between boundaries are DTOs, not internal entities | WARNING |

**Red flags**: Shotgun surgery (one change requires edits in 5+ files across modules), feature envy (module A constantly reaching into module B's internals).

### 4. Abstraction Quality

| Check | Severity if violated |
|-------|---------------------|
| Abstractions represent real domain concepts, not technical plumbing | WARNING |
| No leaky abstractions exposing implementation details | WARNING |
| Interfaces are narrow and cohesive (ISP) | SUGGESTION |
| No premature abstraction (abstraction without 2+ concrete uses) | SUGGESTION |
| Missing abstractions where variation is likely (DB, external APIs, messaging) | WARNING |

### 5. Scalability and Evolution

| Check | Severity if violated |
|-------|---------------------|
| System can scale horizontally without architectural changes | WARNING |
| State is externalized (no in-process state that prevents scaling) | CRITICAL |
| New features can be added without modifying existing core code (OCP) | SUGGESTION |
| Service boundaries align with team/domain boundaries | SUGGESTION |
| Async processing used for work that doesn't need synchronous response | SUGGESTION |

### 6. Error Boundaries

| Check | Severity if violated |
|-------|---------------------|
| Errors don't propagate across architectural boundaries unhandled | CRITICAL |
| Each layer translates errors to its own abstraction level | WARNING |
| Failure in one subsystem doesn't cascade to unrelated subsystems | CRITICAL |

## Output Format

Present findings as a structured report:

```markdown
## Architecture Review: [Component/Service Name]

**Scope**: [What was reviewed — files, modules, PRs]
**Overall**: [PASS | PASS WITH WARNINGS | FAIL]

### Findings

#### [CRITICAL] <Title>
**Location**: `path/to/file.ts:42`
**Issue**: <What's wrong>
**Impact**: <Why it matters>
**Fix**: <Specific recommendation>

#### [WARNING] <Title>
...

#### [SUGGESTION] <Title>
...

### Summary
| Severity | Count |
|----------|-------|
| Critical | N |
| Warning | N |
| Suggestion | N |

### Recommendations
1. <Prioritized action item>
2. ...
```

## Anti-Patterns to Flag

| Anti-Pattern | Signal | Severity |
|--------------|--------|----------|
| God Service | Service >500 LOC with many unrelated methods | WARNING |
| Distributed Monolith | Services tightly coupled via synchronous chains | CRITICAL |
| Anemic Domain Model | Entities are data bags, all logic in services | WARNING |
| Big Ball of Mud | No discernible layer structure | CRITICAL |
| Golden Hammer | Same pattern forced everywhere regardless of fit | SUGGESTION |
| Unnecessary Indirection | Interfaces with single implementation and no test doubles | SUGGESTION |
