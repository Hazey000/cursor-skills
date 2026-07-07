---
name: execution-feature-flags
description: >-
  Implement feature flags safely covering flag types, targeting strategies,
  lifecycle management, and cleanup. Use when implementing progressive delivery,
  A/B testing, or decoupling deployment from release.
---

# Feature Flags

## Purpose

Step-by-step procedure for implementing and managing feature flags.

## Flag Types

| Type | Lifespan | Purpose | Example |
|------|----------|---------|---------|
| Release | Days-weeks | Gate incomplete features | New checkout flow |
| Experiment | Weeks | A/B test variants | Button color test |
| Ops | Permanent | Runtime control | Kill switch, rate limit |
| Permission | Permanent | Entitlement gating | Premium features |

## Workflow

### Step 1: Define the Flag

- Name: descriptive, namespaced (`payments.new-checkout-flow`)
- Type: boolean, string (variant), number
- Default: off/control (safe fallback)
- Owner: who is responsible for cleanup?

### Step 2: Implement with Clean Boundaries

- Flag check at the highest level possible (routing, not deep in logic)
- Keep old and new code paths separate (don't interleave)
- Ensure both paths are tested

### Step 3: Progressive Rollout

1. Internal users (dogfood)
2. Small % of external users (1%, 5%, 10%)
3. Expand while monitoring metrics
4. 100% → remove flag

### Step 4: Cleanup (Critical)

After full rollout and validation:
- Remove the flag check
- Remove the old code path
- Delete the flag from the management system
- Track cleanup as tech debt if not done immediately

## Anti-Patterns

- Flags that live forever (technical debt accumulation)
- Nested flag dependencies (combinatorial explosion)
- Flags deep in business logic (hard to reason about)
- No monitoring of flag states in production
- Testing only the "on" path

## Checklist

- [ ] Flag has a defined owner and cleanup date
- [ ] Default value is the safe/existing behavior
- [ ] Both paths have test coverage
- [ ] Monitoring tracks per-flag-state metrics
- [ ] Cleanup ticket created at flag creation time

## Related Skills

- `deployment/knowledge-release-strategies` — flags as release strategy
- `testing/knowledge-test-strategy` — testing flagged code
