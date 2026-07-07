---
name: review-performance
description: >-
  Identify performance issues including N+1 queries, missing caching, memory
  leaks, blocking operations, and unoptimized payloads. Checklist organized
  by layer (database, backend, frontend, network).
---

# Performance Review

Evaluate code for performance issues across all layers. Focus on measurable impact, not micro-optimizations.

**Cross-reference**: `performance/knowledge-optimization-principles` for measurement-first approach and Amdahl's law.

## When to Use

- Before merging code that handles data-intensive operations
- When users report slowness or timeouts
- When adding new queries, API calls, or rendering logic
- Before load testing or production launch

## Severity Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Will degrade under load or cause outages — fix before merge |
| **WARNING** | Noticeable impact on user experience or resource usage — fix soon |
| **SUGGESTION** | Optimization opportunity — measure before investing effort |

## Review Checklist by Layer

### Database Layer

| Check | Severity if violated |
|-------|---------------------|
| No N+1 query patterns (loop issuing one query per item) | CRITICAL |
| Queries use appropriate indexes (check with `EXPLAIN`) | CRITICAL |
| `SELECT *` avoided — only fetch needed columns | WARNING |
| Pagination implemented for unbounded result sets | CRITICAL |
| Bulk operations used instead of row-by-row inserts/updates | WARNING |
| Transactions are short-lived, no user interaction inside transactions | CRITICAL |
| Connection pooling configured with appropriate limits | WARNING |
| Read replicas used for read-heavy, eventually-consistent queries | SUGGESTION |
| Expensive aggregations pre-computed or cached | SUGGESTION |
| Large text/blob columns excluded from default queries | WARNING |

**Common N+1 patterns**:
```
# BAD: N+1
users = db.query("SELECT * FROM users")
for user in users:
    orders = db.query("SELECT * FROM orders WHERE user_id = ?", user.id)

# GOOD: Join or batch
users_with_orders = db.query("""
    SELECT u.*, o.* FROM users u
    LEFT JOIN orders o ON o.user_id = u.id
""")
```

### Backend Layer

| Check | Severity if violated |
|-------|---------------------|
| Blocking I/O not performed on request-handling threads | CRITICAL |
| CPU-intensive work offloaded to background jobs/workers | WARNING |
| Caching applied for data that's read often, changes rarely | WARNING |
| Cache invalidation strategy is correct (no stale reads in critical paths) | WARNING |
| Response payloads contain only data the client needs | WARNING |
| Unnecessary serialization/deserialization avoided | SUGGESTION |
| No redundant DB calls in single request path (same data fetched twice) | WARNING |
| Streaming used for large file/data transfers | WARNING |
| Timeouts set on all external calls (HTTP, DB, queues) | CRITICAL |
| Memory-proportional-to-input patterns avoided (loading entire dataset into memory) | CRITICAL |

### Frontend Layer

| Check | Severity if violated |
|-------|---------------------|
| Lists/tables use virtualization for large datasets (>100 items) | WARNING |
| Images are lazy-loaded and appropriately sized | WARNING |
| Bundle splitting / code splitting for large applications | WARNING |
| No unnecessary re-renders (React: proper key usage, memoization for expensive computations) | WARNING |
| Expensive computations not run on every render | WARNING |
| Debouncing/throttling on high-frequency events (scroll, resize, input) | WARNING |
| Assets compressed (gzip/brotli) and minified | SUGGESTION |
| No memory leaks (event listeners, timers, subscriptions cleaned up) | CRITICAL |
| Web fonts subset or system fonts used | SUGGESTION |

### Network Layer

| Check | Severity if violated |
|-------|---------------------|
| API calls batched where possible (no waterfall of sequential requests) | WARNING |
| HTTP caching headers set correctly (Cache-Control, ETag) | WARNING |
| Compression enabled for API responses | SUGGESTION |
| No over-fetching (requesting more data than displayed) | WARNING |
| No under-fetching (multiple round trips for one view) | WARNING |
| CDN used for static assets | SUGGESTION |
| Connection keep-alive enabled | SUGGESTION |
| GraphQL: no unbounded depth or complexity | CRITICAL |

## Common Performance Anti-Patterns

| Anti-Pattern | Signal | Severity |
|--------------|--------|----------|
| Unbounded query | `SELECT` without `LIMIT` or pagination | CRITICAL |
| Synchronous cascade | Endpoint makes 5+ sequential external calls | CRITICAL |
| Cache stampede | Popular cache key expires, all requests hit DB simultaneously | WARNING |
| Premature optimization | Complex caching for data accessed 10 times/day | SUGGESTION |
| Chatty interface | 20 API calls to render one page | WARNING |
| Fat payload | Sending 2MB JSON when client needs 5 fields | WARNING |
| Missing timeout | External HTTP call with no timeout (blocks thread indefinitely) | CRITICAL |

## Output Format

```markdown
## Performance Review: [Component/Service Name]

**Scope**: [What was reviewed]
**Overall**: [PASS | PASS WITH WARNINGS | FAIL]

### Findings by Layer

#### Database
##### [CRITICAL] N+1 Query in Order List
**Location**: `src/orders/list.ts:34-42`
**Issue**: Each order triggers a separate query for customer details
**Impact**: 100 orders = 101 queries, ~2s response time
**Fix**: Use JOIN or batch `WHERE customer_id IN (...)`
**Estimated improvement**: ~95% reduction in query count

...

### Summary
| Layer | Critical | Warning | Suggestion |
|-------|----------|---------|------------|
| Database | 1 | 0 | 1 |
| Backend | 0 | 2 | 0 |
| Frontend | 0 | 1 | 1 |
| Network | 0 | 0 | 1 |
| **Total** | **1** | **3** | **3** |

### Recommendations
1. <Prioritized by impact>
2. ...
```
