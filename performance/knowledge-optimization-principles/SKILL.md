---
name: knowledge-optimization-principles
description: >-
  Performance optimization philosophy covering measurement-first approach,
  Amdahl's law, premature optimization avoidance, and systematic bottleneck
  identification. Use when deciding whether to optimize, where to optimize,
  or reviewing performance-related changes.
---

# Optimization Principles

## Purpose

Decision framework for when and where to optimize, avoiding premature optimization while not ignoring real performance issues.

## Core Rules

1. **Measure first**: Never optimize without profiling data
2. **Optimize the bottleneck**: 10x improvement on a non-bottleneck yields nothing
3. **Set a target**: "Fast enough" needs a number (latency budget, throughput requirement)
4. **Amdahl's Law**: Speedup limited by the non-optimized portion

## Optimization Decision Framework

| Question | If yes | If no |
|----------|--------|-------|
| Is there a measured performance problem? | Continue | Stop, don't optimize |
| Do you have a specific target? | Continue | Define one first |
| Have you profiled to find the bottleneck? | Continue | Profile first |
| Is the fix worth the complexity? | Implement | Accept current performance |

## Where Performance Usually Hides

| Layer | Common culprits |
|-------|-----------------|
| Network | Too many round trips, large payloads, no compression |
| Database | N+1 queries, missing indexes, full table scans |
| Application | Unnecessary computation, blocking I/O, memory allocation |
| Frontend | Large bundles, unoptimized images, layout thrash |

## When to Optimize

- **During design**: Choose appropriate data structures and algorithms
- **At boundaries**: Pagination, payload size, caching decisions
- **When measured**: Profiling shows specific bottleneck
- **At scale thresholds**: Load testing reveals limits

## When NOT to Optimize

- Before measuring (premature optimization)
- Low-traffic code paths
- During initial development (except obvious algorithm choices)
- When clarity would suffer significantly

## Related Skills

- `performance/execution-profiling` — how to find bottlenecks
- `performance/execution-load-testing` — testing under load
- `performance/knowledge-caching-patterns` — caching as optimization
- `reviewers/review-performance` — performance review criteria
