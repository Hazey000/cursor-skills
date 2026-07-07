---
name: knowledge-release-strategies
description: >-
  Deployment and release strategies covering blue-green, canary, rolling,
  feature flags, and progressive delivery. Decision framework for choosing
  the right strategy. Use when planning releases, designing deployment
  infrastructure, or choosing rollback mechanisms.
---

# Release Strategies

## Purpose

Decision framework for choosing how to release changes to production safely.

## Strategy Comparison

| Strategy | Risk | Complexity | Rollback speed | Resource cost |
|----------|------|------------|----------------|---------------|
| Big bang | High | Low | Slow (redeploy) | Low |
| Rolling | Medium | Medium | Medium (new rollout) | Low |
| Blue-green | Low | Medium | Instant (swap) | 2x during deploy |
| Canary | Low | High | Fast (route away) | Low extra |
| Feature flags | Lowest | High (code) | Instant (toggle) | None extra |

## Decision Framework

| Situation | Recommended strategy |
|-----------|---------------------|
| Simple service, low traffic | Rolling deployment |
| Need instant rollback | Blue-green |
| Risky change, gradual validation | Canary |
| Decouple deploy from release | Feature flags |
| Database migration included | Blue-green + backward-compatible migration |
| User-facing feature, gradual rollout | Feature flags + canary |

## Strategy Details

### Rolling
Replace instances one-by-one. Old and new versions coexist briefly.
- Requires: Backward-compatible changes
- Rollback: New rolling update with old version

### Blue-Green
Two identical environments. Route traffic from blue (old) to green (new).
- Requires: Duplicate infrastructure, database compatibility
- Rollback: Route back to blue

### Canary
Route small percentage of traffic to new version. Increase if healthy.
- Requires: Traffic splitting, automated health metrics
- Rollback: Route 100% back to old version

### Feature Flags
Deploy code dark (disabled), enable progressively per user segment.
- Requires: Flag management system, cleanup discipline
- Rollback: Toggle flag off

## Common Pitfalls

- No defined rollback procedure (define it before deploying)
- Canary without automated health checks (human-gated is too slow)
- Feature flag debt (flags that live forever)
- Blue-green with shared database state (schema changes break swap)

## Related Skills

- `deployment/knowledge-cicd-principles` — release in the pipeline context
- `deployment/execution-feature-flags` — implementing feature flags
- `deployment/execution-cicd-pipeline` — pipeline that supports these strategies
