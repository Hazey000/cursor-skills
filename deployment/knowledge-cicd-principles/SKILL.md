---
name: knowledge-cicd-principles
description: >-
  Continuous integration and continuous delivery philosophy. Covers CI/CD
  principles, pipeline stages, deployment strategies, and the path from
  commit to production. Use when designing CI/CD pipelines or reviewing
  deployment processes.
---

# CI/CD Principles

## Purpose

Foundational principles for designing reliable continuous integration and delivery systems.

## Core Philosophy

- **Continuous Integration**: Every developer integrates frequently (at least daily), each integration verified by automated build + test
- **Continuous Delivery**: Software is always in a releasable state; deployment is a business decision
- **Continuous Deployment**: Every change that passes the pipeline goes to production automatically

## Pipeline Stages

| Stage | Speed | Purpose | Fail action |
|-------|-------|---------|-------------|
| Lint/Format | Seconds | Code style consistency | Block merge |
| Unit tests | Seconds-minutes | Logic correctness | Block merge |
| Build | Minutes | Compilable, packageable | Block merge |
| Integration tests | Minutes | Component interactions | Block merge |
| Security scan | Minutes | Vulnerability detection | Block merge (critical) / warn (medium) |
| Deploy to staging | Minutes | Environment verification | Block production deploy |
| E2E/Smoke tests | Minutes | Critical path validation | Block production deploy |
| Deploy to production | Minutes | Release | Automatic rollback on health failure |

## Key Principles

1. **Fast feedback**: Cheapest/fastest checks first
2. **Deterministic**: Same commit → same result
3. **Trunk-based friendly**: Short-lived branches, frequent merges
4. **Immutable artifacts**: Build once, deploy same artifact everywhere
5. **Environment parity**: Same infra/config shape across envs

## Anti-Patterns

- Manual steps in the deployment process
- "Works on my machine" (environment parity violation)
- Long-running feature branches (merge hell)
- Testing only in production-like environments (slow feedback)
- No rollback plan

## Related Skills

- `deployment/execution-cicd-pipeline` — implementing a pipeline
- `deployment/execution-git-workflow` — branch strategy feeding CI
- `deployment/knowledge-release-strategies` — deployment patterns
- `infrastructure/knowledge-twelve-factor` — Factor 5 (build, release, run)
