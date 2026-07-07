---
name: knowledge-component-design
description: >-
  Component composition patterns, prop design, state boundaries, and reusability
  principles. Framework-agnostic patterns for building maintainable UI component
  trees. Use when designing component hierarchies, deciding component boundaries,
  or reviewing frontend architecture.
---

# Component Design

## Purpose

Decision frameworks for decomposing UIs into well-bounded, reusable components.

## Key Principles

- **Single responsibility**: One component, one reason to change
- **Composition over configuration**: Small composable components over large configurable ones
- **Prop drilling threshold**: If props pass through 3+ levels without being used, introduce context or composition
- **Smart vs Dumb**: Container components (data fetching, state) vs presentational components (rendering, styling)

## Component Boundary Decision Framework

Split a component when:
1. It has multiple independent concerns (data + presentation + interaction)
2. Part of it is reusable elsewhere
3. It's too large to reason about (>150-200 lines is a smell)
4. Different parts change at different rates

Keep together when:
1. Splitting would require passing 5+ props to the child
2. The "components" would only ever be used together
3. The logic is genuinely cohesive

## Prop Design Principles

- Prefer primitive props over object props for leaf components
- Boolean props should default to `false`
- Callbacks should be named `onAction` (e.g., `onSubmit`, `onChange`)
- Avoid prop explosion — if >7 props, consider composition or compound component pattern

## Anti-Patterns

- **God components**: 500+ line components that do everything
- **Prop drilling**: Passing props through many layers untouched
- **Premature abstraction**: Creating "reusable" components before the second use case
- **Inheritance-based components**: Prefer composition

## Related Skills

- `frontend/knowledge-state-management` — state boundaries between components
- `frontend/knowledge-accessibility` — accessible component patterns
- `principles/knowledge-separation-of-concerns` — general boundary principles
