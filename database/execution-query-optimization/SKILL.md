---
name: execution-query-optimization
description: >-
  Identify and fix slow database queries. Covers EXPLAIN analysis, common
  query anti-patterns, N+1 detection, and optimization techniques. Use when
  debugging slow queries or reviewing database access patterns.
---

# Query Optimization

## Purpose

Step-by-step procedure for finding and fixing slow queries.

## Workflow

### Step 1: Identify Slow Queries

Sources:
- Slow query log (queries exceeding threshold)
- APM tool (database time in request traces)
- `pg_stat_statements` (cumulative query stats)
- Application logs with query timing

### Step 2: Analyze with EXPLAIN

Read EXPLAIN ANALYZE output for:
- **Seq Scan** on large tables → missing index
- **Nested Loop** with large outer set → consider Hash Join
- **Sort** with high row count → add index for ORDER BY
- **Rows estimated vs actual** diverge → stale statistics

### Step 3: Common Fixes

| Problem | Fix |
|---------|-----|
| Sequential scan | Add appropriate index |
| N+1 queries | Batch/eager load (JOIN or IN clause) |
| SELECT * | Select only needed columns |
| Large OFFSET | Keyset pagination (WHERE id > last_seen) |
| Expensive subquery | Rewrite as JOIN or CTE |
| Missing WHERE clause on large table | Add filter, use partial index |
| OR conditions preventing index use | UNION of indexed queries |

### Step 4: Verify

- Re-run EXPLAIN ANALYZE after fix
- Compare execution time
- Monitor in production (may behave differently with real data distribution)

## N+1 Detection

**Pattern**: Loop that executes one query per item.

**Fix**: 
- Eager loading (JOIN in the original query)
- Batch loading (WHERE id IN (...) for the batch)
- DataLoader pattern (batch + cache within request)

## Anti-Patterns

- `SELECT *` when only 2 columns needed
- `LIKE '%prefix%'` (can't use B-tree index; need GIN trigram)
- Pagination with OFFSET on large datasets
- Implicit type casts in WHERE clause (prevents index use)
- Functions on indexed columns in WHERE (use functional index or rewrite)

## Related Skills

- `database/knowledge-indexing-strategy` — designing indexes
- `performance/execution-profiling` — systematic performance investigation
- `reviewers/review-performance` — query review criteria
