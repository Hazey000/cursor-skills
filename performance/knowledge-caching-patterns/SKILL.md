---
name: knowledge-caching-patterns
description: >-
  Caching theory and patterns covering cache topologies, consistency models,
  eviction strategies, and invalidation approaches. Use when designing caching
  systems, choosing cache invalidation strategies, or debugging cache-related
  issues.
---

# Caching Patterns

## Purpose

Deep understanding of caching trade-offs and patterns beyond the basic cache-aside.

## The Fundamental Trade-off

Caching trades **consistency** for **latency/throughput**. Every caching decision is a statement about acceptable staleness.

## Consistency Models

| Model | Guarantee | Implementation |
|-------|-----------|---------------|
| Strong | Read always returns latest write | Write-through + synchronous invalidation |
| Bounded staleness | Data at most N seconds old | TTL-based |
| Eventual | Will converge, no time guarantee | Event-driven invalidation |

## Eviction Strategies

| Strategy | Evicts | Best for |
|----------|--------|----------|
| LRU (Least Recently Used) | Oldest accessed item | General purpose |
| LFU (Least Frequently Used) | Least popular item | Stable access patterns |
| TTL (Time To Live) | Expired items | Time-sensitive data |
| FIFO (First In First Out) | Oldest inserted | Simple, predictable |

## Multi-Level Caching

```
[Browser Cache] → [CDN] → [App-level Cache] → [Database Cache] → [Database]
```

Each level adds latency reduction but complicates invalidation.

**Rule**: Invalidation complexity grows multiplicatively with cache layers. Add layers deliberately.

## Cache Warming

Pre-populate cache before traffic arrives:
- After deployment (known-popular keys)
- After cache flush (replay recent queries)
- Before peak traffic (scheduled warm-up)

## Anti-Patterns

- Caching everything (memory waste, invalidation complexity)
- Cache without TTL (stale forever on missed invalidation)
- Cache key collisions (shared cache between tenants)
- Relying on cache for correctness (cache is optimization, not storage)

## Related Skills

- `backend/execution-caching-strategy` — implementing caching
- `performance/knowledge-optimization-principles` — caching as optimization
- `infrastructure/knowledge-networking` — CDN as cache layer
