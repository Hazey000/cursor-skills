---
name: execution-caching-strategy
description: >-
  Choose and implement caching layers for backend systems. Covers cache
  placement, invalidation strategies, cache-aside pattern, TTL design, and
  cache stampede prevention. Use when adding caching to a system or debugging
  cache-related issues.
---

# Caching Strategy

## Purpose

Step-by-step procedure for choosing and implementing effective caching.

## Workflow

### Step 1: Identify What to Cache

Good cache candidates:
- Expensive computations with stable inputs
- Frequently read, infrequently written data
- External API responses with acceptable staleness
- Database queries with predictable access patterns

Poor cache candidates:
- Highly personalized data (low hit rate)
- Data that must always be current
- Write-heavy data (invalidation overhead)

### Step 2: Choose Cache Placement

| Layer | Latency | Scope | Example |
|-------|---------|-------|---------|
| In-process | ~ns | Single instance | HashMap, LRU cache |
| Distributed | ~ms | All instances | Redis, Memcached |
| CDN | ~ms | Global | Static assets, API responses |
| Browser | 0 | Single user | HTTP cache headers |

### Step 3: Choose Strategy

| Pattern | How it works | Best for |
|---------|-------------|----------|
| Cache-aside | App checks cache, fills on miss | General purpose |
| Read-through | Cache fetches on miss | Simplify app code |
| Write-through | Write to cache and DB together | Strong consistency needs |
| Write-behind | Write to cache, async to DB | Write-heavy workloads |

### Step 4: Design Invalidation

**The two hard problems**: cache invalidation and naming things.

| Approach | Consistency | Complexity |
|----------|-------------|------------|
| TTL-based | Eventual (bounded staleness) | Low |
| Event-based | Near-real-time | Medium |
| Write-through | Strong | Medium |
| Manual purge | On-demand | Low (but error-prone) |

### Step 5: Prevent Stampedes

When a popular cache entry expires, many requests hit the backend simultaneously.

Mitigations:
- **Lock/single-flight**: Only one request refreshes, others wait
- **Early refresh**: Refresh before TTL expires (probabilistic early expiration)
- **Stale-while-revalidate**: Serve stale data while refreshing in background

## Checklist

- [ ] Cache hit rate monitored (target: >80% for general caches)
- [ ] TTL set with acceptable staleness tolerance
- [ ] Cache stampede prevention for hot keys
- [ ] Graceful degradation when cache is unavailable
- [ ] Cache size bounded (eviction policy: LRU, LFU)
- [ ] Serialization format chosen (avoid language-specific formats)

## Related Skills

- `performance/knowledge-caching-patterns` — deeper caching theory
- `performance/knowledge-optimization-principles` — measure before caching
- `backend/knowledge-concurrency` — cache access under concurrency
