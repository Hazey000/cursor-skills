---
name: knowledge-schema-design
description: >-
  Relational database schema design principles covering normalization,
  denormalization trade-offs, naming conventions, constraints, and relationship
  modeling. Use when designing new schemas, reviewing database designs, or
  deciding on normalization level.
---

# Schema Design

## Purpose

Decision frameworks for designing relational schemas that balance data integrity with query performance.

## Normalization Decision Framework

| Normal Form | What it prevents | Cost of enforcement |
|-------------|-----------------|---------------------|
| 1NF | Repeating groups, non-atomic values | Minimal |
| 2NF | Partial dependencies (composite keys) | Low |
| 3NF | Transitive dependencies | Low-medium |
| BCNF | Non-trivial FD violations | Medium |
| Beyond 3NF | Rarely needed | High complexity |

**Default**: Design to 3NF, denormalize deliberately with documented rationale.

## When to Denormalize

| Signal | Denormalization technique | Trade-off |
|--------|--------------------------|-----------|
| Expensive joins on hot path | Materialized view or duplicated column | Write complexity |
| Read-heavy reporting | Pre-aggregated tables | Storage + staleness |
| Hierarchical data display | Adjacency list → nested set or materialized path | Write complexity |
| Multi-tenant filtering | Tenant ID on every table | Slightly redundant |

## Naming Conventions

- Tables: `plural_snake_case` (e.g., `order_items`)
- Columns: `singular_snake_case` (e.g., `created_at`)
- Primary keys: `id` or `table_singular_id`
- Foreign keys: `referenced_table_singular_id` (e.g., `user_id`)
- Booleans: `is_` or `has_` prefix (e.g., `is_active`)
- Timestamps: `_at` suffix (e.g., `updated_at`)

## Constraints Checklist

- [ ] Primary key on every table
- [ ] Foreign keys for all relationships
- [ ] NOT NULL on columns that should never be null
- [ ] UNIQUE constraints on natural keys
- [ ] CHECK constraints for value ranges/enums
- [ ] Default values where semantically clear

## Anti-Patterns

- **EAV (Entity-Attribute-Value)**: Use JSON columns or proper modeling instead
- **Polymorphic associations**: Foreign key to "any table" — use separate FKs or shared interface table
- **Soft deletes everywhere**: Complicates all queries — prefer event log or archive table
- **Stringly-typed data**: Dates as strings, enums as free text

## Related Skills

- `database/knowledge-indexing-strategy` — indexes for the schema
- `database/knowledge-data-modeling` — conceptual to physical modeling
- `database/execution-migration-workflow` — evolving schemas safely
- `reviewers/review-database` — schema review criteria
