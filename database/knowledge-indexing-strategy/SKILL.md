---
name: knowledge-indexing-strategy
description: >-
  Database indexing principles covering index types, composite key design,
  covering indexes, and query plan analysis. Decision framework for when and
  what to index. Use when optimizing queries, designing indexes for new
  tables, or debugging slow queries.
---

# Indexing Strategy

## Purpose

Decision framework for designing indexes that serve query patterns without over-indexing.

## Index Decision Framework

**Index when**:
- Column appears in WHERE clauses frequently
- Column used in JOIN conditions
- Column used in ORDER BY with query filters
- Column has high cardinality (many distinct values)

**Don't index when**:
- Table is small (<1000 rows) — full scan is fine
- Column has very low cardinality (e.g., boolean on large table with 50/50 split)
- Column is frequently updated (index maintenance cost)
- Write performance is critical and reads are infrequent

## Index Types

| Type | Use for | Example |
|------|---------|---------|
| B-tree (default) | Equality, range, sorting | Most columns |
| Hash | Equality only (no range) | Exact lookups |
| GIN | Array/JSONB containment, full-text | Tags, document search |
| GiST | Geometric, range types, full-text | Geo queries |
| Partial | Subset of rows matching condition | `WHERE is_active = true` |
| Covering | Include all columns query needs | Avoid table lookup |

## Composite Index Design

**Column order matters**: leftmost prefix rule.

Index on `(a, b, c)` supports:
- WHERE a = ?
- WHERE a = ? AND b = ?
- WHERE a = ? AND b = ? AND c = ?
- WHERE a = ? ORDER BY b

Does NOT support:
- WHERE b = ? (skips first column)
- WHERE a = ? AND c = ? (gap in prefix)

**Rule**: Place equality columns first, range/sort columns last.

## Anti-Patterns

- Index on every column (write overhead, storage waste)
- Duplicate indexes (index on `(a)` when `(a, b)` already exists)
- Indexes never used by any query
- Missing index on foreign keys (slow joins and cascading deletes)

## Monitoring

- Identify unused indexes (pg_stat_user_indexes)
- Identify missing indexes (pg_stat_user_tables sequential scans)
- Monitor index bloat
- Track query plan changes after index additions

## Related Skills

- `database/knowledge-schema-design` — schema that indexes well
- `database/execution-query-optimization` — using indexes effectively
- `performance/execution-profiling` — finding slow queries
