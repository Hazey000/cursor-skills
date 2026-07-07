---
name: knowledge-state-management
description: >-
  State management decision framework covering local vs global state, state
  machines, derived state, and server state. Framework-agnostic principles for
  managing UI state complexity. Use when choosing state management approaches
  or debugging state-related bugs.
---

# State Management

## Purpose

Decision framework for where state lives, how it flows, and when to reach for global state management.

## State Categories

| Category | Scope | Examples | Typical solution |
|----------|-------|----------|-----------------|
| UI state | Single component | Open/closed, hover, focus | Local state |
| Form state | Form boundary | Input values, validation errors | Form library or local |
| Server state | Cached API data | User profile, list results | Server state library (query cache) |
| App state | Cross-component | Auth, theme, locale | Context or global store |
| URL state | Navigation | Filters, pagination, active tab | URL/router |

## Decision Framework

1. **Can this be derived from other state?** → Don't store it, compute it
2. **Does only one component need it?** → Local state
3. **Is it server data?** → Server state library with cache
4. **Is it in the URL?** → URL state
5. **Do many unrelated components need it?** → Global state (minimal)

## Key Principles

- State should live as close to where it's used as possible
- Prefer derived/computed state over stored duplicates
- Server state is not application state — treat differently
- URL is state too — use it for shareable/bookmarkable state
- State machines prevent impossible states

## Anti-Patterns

- Storing derived values (instead of computing them)
- Putting everything in global state
- Syncing the same data in multiple places
- Using global state for form values

## Related Skills

- `frontend/knowledge-component-design` — component boundaries affect state placement
- `architecture/knowledge-event-driven` — event patterns applicable to frontend
