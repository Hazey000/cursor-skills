---
name: review-database
description: >-
  Review database schema design, index coverage, query patterns, migration
  safety, naming conventions, and data integrity constraints. Covers relational
  and document databases.
---

# Database Review

Evaluate schema design, query patterns, and migration safety for correctness, performance, and operability.

**Cross-reference**: `database/knowledge-schema-design` for normalization, denormalization trade-offs, and modeling patterns.

## When to Use

- Before merging schema migrations
- When adding new tables, collections, or indexes
- When reviewing query patterns in application code
- Before production deployment of data model changes

## Severity Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Data loss risk, corruption, or severe performance issue — fix before merge |
| **WARNING** | Design issue that will cause maintenance or performance pain — fix soon |
| **SUGGESTION** | Improvement for clarity or future-proofing — consider when practical |

## Review Checklist

### 1. Schema Design

| Check | Severity if violated |
|-------|---------------------|
| Primary keys defined on all tables | CRITICAL |
| Foreign keys have appropriate constraints (ON DELETE behavior) | CRITICAL |
| NOT NULL constraints on fields that must always have values | WARNING |
| UNIQUE constraints on natural keys (email, username, etc.) | WARNING |
| CHECK constraints for domain-restricted values (status enums, positive amounts) | WARNING |
| No overly wide columns (VARCHAR(4000) for a country code) | SUGGESTION |
| Normalization appropriate for access patterns (3NF default, denormalize with justification) | WARNING |
| Audit columns present where needed (created_at, updated_at, created_by) | SUGGESTION |
| Soft delete pattern consistent if used (deleted_at, not is_deleted boolean) | SUGGESTION |

### 2. Naming Conventions

| Check | Severity if violated |
|-------|---------------------|
| Table names are consistent (all plural or all singular — pick one) | WARNING |
| Column names use snake_case (or the project's established convention) | WARNING |
| Join tables named `{table_a}_{table_b}` in alphabetical order | SUGGESTION |
| Boolean columns use `is_`, `has_`, `can_` prefix | SUGGESTION |
| Foreign key columns named `{referenced_table}_id` | WARNING |
| Index names follow pattern: `idx_{table}_{columns}` | SUGGESTION |
| No reserved word conflicts (order, user, group, index) | WARNING |

### 3. Index Coverage

| Check | Severity if violated |
|-------|---------------------|
| Columns in WHERE clauses of frequent queries are indexed | CRITICAL |
| Foreign key columns are indexed | WARNING |
| Composite indexes have columns in correct order (highest cardinality first for equality, range column last) | WARNING |
| No redundant indexes (index on `(a)` when `(a, b)` exists) | WARNING |
| No over-indexing (indexes on rarely-queried columns increase write cost) | SUGGESTION |
| Partial indexes used for filtered queries (`WHERE status = 'active'`) | SUGGESTION |
| Covering indexes considered for read-heavy queries | SUGGESTION |
| Full-text search uses appropriate index type (GIN, FTS) not LIKE '%term%' | WARNING |

### 4. Query Patterns

| Check | Severity if violated |
|-------|---------------------|
| No N+1 patterns (see review-performance) | CRITICAL |
| SELECT specifies columns, not `SELECT *` | WARNING |
| Unbounded queries have LIMIT/pagination | CRITICAL |
| Aggregations on large tables are pre-computed or use materialized views | WARNING |
| No implicit type coercion in WHERE clauses (comparing string to int) | WARNING |
| No functions on indexed columns in WHERE (`WHERE LOWER(email) = ...` — use functional index) | WARNING |
| Transactions used for multi-statement consistency | CRITICAL |
| No long-running transactions (holding locks) | CRITICAL |

### 5. Migration Safety

| Check | Severity if violated |
|-------|---------------------|
| Migration is backward-compatible with current running code | CRITICAL |
| Column renames use add-new/migrate/drop-old, not ALTER RENAME | CRITICAL |
| Adding NOT NULL column includes DEFAULT or is nullable first | CRITICAL |
| Large table migrations use batching (not one massive UPDATE) | CRITICAL |
| Migrations are idempotent (safe to re-run) | WARNING |
| Rollback migration provided and tested | WARNING |
| Indexes created CONCURRENTLY on production tables | CRITICAL |
| No exclusive table locks during deployment | CRITICAL |
| Data backfill separated from schema change | WARNING |

**Safe migration pattern for adding a required column**:
1. Add column as nullable: `ALTER TABLE t ADD COLUMN c TYPE NULL`
2. Deploy code that writes to new column
3. Backfill existing rows in batches
4. Add NOT NULL constraint: `ALTER TABLE t ALTER COLUMN c SET NOT NULL`

### 6. Data Integrity

| Check | Severity if violated |
|-------|---------------------|
| Referential integrity enforced at DB level, not just application | CRITICAL |
| Orphan records prevented (cascading deletes or restrict) | WARNING |
| Monetary values use DECIMAL/NUMERIC, not FLOAT | CRITICAL |
| Timestamps stored in UTC with timezone info | WARNING |
| UUID vs auto-increment choice is intentional | SUGGESTION |
| Enum values stored as constrained types, not unchecked strings | WARNING |

### 7. Operational Concerns

| Check | Severity if violated |
|-------|---------------------|
| Table estimated to grow beyond 10M rows has partitioning strategy | WARNING |
| Connection pool sized for expected concurrency | WARNING |
| Slow query logging enabled | SUGGESTION |
| Backup and recovery tested for new tables/schemas | WARNING |
| PII columns identified for GDPR/compliance (deletion, export) | WARNING |

## Output Format

```markdown
## Database Review: [Migration/Schema Name]

**Scope**: [Tables, migrations, queries reviewed]
**Overall**: [PASS | PASS WITH WARNINGS | FAIL]

### Findings

#### Schema
##### [CRITICAL] Missing foreign key constraint
**Location**: `migrations/20240115_add_orders.sql:12`
**Issue**: `orders.customer_id` has no FK constraint to `customers.id`
**Impact**: Orphan orders possible, referential integrity broken
**Fix**: `ALTER TABLE orders ADD CONSTRAINT fk_orders_customer FOREIGN KEY (customer_id) REFERENCES customers(id)`

#### Migration Safety
##### [CRITICAL] Non-concurrent index creation
**Location**: `migrations/20240115_add_orders.sql:25`
**Issue**: `CREATE INDEX` without CONCURRENTLY on production table
**Impact**: Exclusive lock, table writes blocked during index build
**Fix**: `CREATE INDEX CONCURRENTLY idx_orders_customer_id ON orders(customer_id)`

...

### Summary
| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Schema | 1 | 1 | 0 |
| Naming | 0 | 0 | 2 |
| Indexes | 0 | 1 | 0 |
| Queries | 1 | 0 | 0 |
| Migrations | 1 | 1 | 0 |
| Integrity | 0 | 0 | 1 |
| Operations | 0 | 1 | 0 |
| **Total** | **3** | **4** | **3** |

### Recommendations
1. <Prioritized by data-loss risk>
2. ...
```
