---
name: knowledge-twelve-factor
description: >-
  Twelve-Factor App methodology for building cloud-native services. Covers all
  12 factors with modern interpretations and practical decision guidance. Use
  when designing services for cloud deployment, containerization, or reviewing
  application portability.
---

# Twelve-Factor App

## Purpose

Design applications that are portable, scalable, and suitable for modern cloud platforms.

## The Twelve Factors

| # | Factor | Principle | Modern interpretation |
|---|--------|-----------|---------------------|
| 1 | Codebase | One codebase, many deploys | Monorepo or single repo per service |
| 2 | Dependencies | Explicitly declare and isolate | Lock files, container images |
| 3 | Config | Store in environment | Env vars, config maps, secrets managers |
| 4 | Backing services | Treat as attached resources | Connection strings, service discovery |
| 5 | Build, release, run | Strict separation of stages | CI/CD pipeline stages |
| 6 | Processes | Stateless processes | No sticky sessions, external state stores |
| 7 | Port binding | Export services via port | Container port exposure |
| 8 | Concurrency | Scale via process model | Horizontal pod autoscaling |
| 9 | Disposability | Fast startup, graceful shutdown | Container orchestration, health checks |
| 10 | Dev/prod parity | Keep environments similar | Containers, IaC, feature flags |
| 11 | Logs | Treat as event streams | stdout/stderr, log aggregation |
| 12 | Admin processes | Run as one-off processes | Kubernetes Jobs, migration containers |

## Most Commonly Violated

1. **Config in code** (Factor 3): Hardcoded URLs, credentials in source
2. **Stateful processes** (Factor 6): In-memory sessions, local file storage
3. **Dev/prod parity** (Factor 10): "Works on my machine" problems
4. **Slow startup** (Factor 9): Heavy initialization blocking readiness

## When to Bend the Rules

- Factor 6 (stateless): Acceptable for single-instance tools, CLIs, PoCs
- Factor 8 (concurrency): Vertical scaling is fine when simpler and sufficient
- Factor 11 (logs): Structured logging to stdout is the minimum; file-based logs acceptable for local dev

## Related Skills

- `infrastructure/knowledge-cloud-architecture` — cloud patterns built on 12-factor
- `infrastructure/execution-containerization` — containers enforce many factors
- `deployment/execution-cicd-pipeline` — factor 5 in practice
