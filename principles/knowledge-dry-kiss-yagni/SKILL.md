---
name: knowledge-dry-kiss-yagni
description: >-
  DRY, KISS, and YAGNI principles with trade-off frameworks for when they
  conflict. Covers when duplication is acceptable, when simplicity trumps
  abstraction, and when to defer features. Use when making design trade-off
  decisions or reviewing code for unnecessary complexity.
---

# DRY, KISS, YAGNI

## Purpose

These three principles frequently tension against each other. This skill provides decision frameworks for navigating those tensions.

## DRY (Don't Repeat Yourself)

**Rule**: Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

**What DRY actually means**:
- DRY is about knowledge duplication, not code duplication
- Two identical code blocks that change for different reasons are NOT a DRY violation
- A business rule expressed in both code and documentation IS a DRY violation

**When duplication is acceptable**:
- Two services that happen to have similar validation but serve different bounded contexts
- Test code that repeats setup for clarity over shared fixtures
- When the "abstraction" would couple things that should vary independently

**Decision framework**:
1. Would a change to one copy always require changing the other? → Extract
2. Could they diverge independently in the future? → Keep separate
3. Is the shared code expressing the same business rule? → Extract
4. Is it just structural similarity? → Leave it

**Anti-pattern**: "Wrong abstraction" — DRYing up code that looks similar but represents different concepts, creating coupling that makes future changes painful.

## KISS (Keep It Simple, Stupid)

**Rule**: Prefer the simplest solution that works correctly.

**What simple means**:
- Fewer moving parts
- Less indirection
- Easier to trace execution mentally
- Fewer concepts a new reader must learn

**Decision framework**:
1. Can I explain this to a new team member in under 2 minutes?
2. If I remove this abstraction, what breaks?
3. Am I adding complexity for a future scenario that may never happen?
4. Is this pattern solving a problem I actually have?

**Tension with SOLID**: SOLID can push toward more abstractions. Apply KISS as a counterbalance — only add indirection when it solves a real, current problem.

## YAGNI (You Aren't Gonna Need It)

**Rule**: Don't build it until you need it.

**What YAGNI covers**:
- Features no one has asked for yet
- Abstractions for flexibility you haven't needed
- Support for hypothetical future requirements
- Configuration for scenarios that don't exist

**What YAGNI does NOT mean**:
- Skip error handling
- Ignore security
- Write untestable code
- Avoid clean interfaces

**Decision framework**:
1. Is someone asking for this now? → Build it
2. Will this be significantly harder to add later? → Consider building it
3. Am I adding this "just in case"? → Don't build it
4. Is this a reversible decision? → Defer it

## Navigating Tensions

| Tension | Resolution |
|---------|------------|
| DRY vs KISS | If the abstraction is hard to name or understand, accept the duplication |
| DRY vs YAGNI | Don't create a shared library for two usages — wait for three |
| KISS vs Extensibility | Start simple, refactor when the extension point proves needed |
| All three conflict | Favour YAGNI first, KISS second, DRY third |

## The Rule of Three

A pragmatic heuristic that resolves many DRY tensions:
1. First time: just write it
2. Second time: note the duplication, accept it
3. Third time: now extract the abstraction

This ensures you have enough examples to design the right abstraction.

## Related Skills

- `principles/knowledge-solid` — principles that can tension with KISS/YAGNI
- `architecture/knowledge-design-patterns` — patterns are KISS violations unless earned
- `poc/knowledge-poc-methodology` — PoCs maximize YAGNI and KISS
