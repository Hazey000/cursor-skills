---
name: execution-migration-workflow
description: >-
  Safe, reversible database schema migration workflow. Covers migration naming,
  zero-downtime changes, backward compatibility, and rollback strategy. Use
  when evolving database schemas in production systems.
---

# Migration Workflow

## Purpose

Step-by-step procedure for safe schema evolution in production.

## Workflow

### Step 1: Plan the Change

Classify the change:

| Change type | Risk | Approach |
|-------------|------|----------|
| Add column (nullable) | Low | Single migration |
| Add column (NOT NULL) | Medium | Multi-step (add nullable → backfill → add constraint) |
| Rename column | High | Multi-step with dual-write period |
| Drop column | High | Remove code references first, then drop |
| Add index | Medium | CREATE INDEX CONCURRENTLY (non-blocking) |
| Change column type | High | Add new column → migrate data → swap |

### Step 2: Write the Migration

- One logical change per migration file
- Include both `up` and `down` (rollback)
- Name: `YYYYMMDDHHMMSS_descriptive_name`
- Test on a copy of production data

### Step 3: Zero-Downtime Checklist

- [ ] Old code works with new schema (backward compatible)
- [ ] New code works with old schema (forward compatible during deploy)
- [ ] No exclusive locks on large tables
- [ ] Indexes created concurrently
- [ ] Backfill runs as separate step (not in migration transaction)

### Step 4: Execute

1. Run migration in staging with production-like data
2. Verify timing (should complete within maintenance window or be non-blocking)
3. Deploy migration
4. Monitor error rates and query performance
5. Deploy application code that uses new schema

### Step 5: Cleanup (if multi-step)

After confirming new code is stable:
- Drop old columns/tables
- Remove dual-write logic
- Remove feature flags

## Dangerous Operations

| Operation | Safe alternative |
|-----------|-----------------|
| `ALTER TABLE ... ADD COLUMN NOT NULL` | Add nullable → backfill → add constraint |
| `CREATE INDEX` | `CREATE INDEX CONCURRENTLY` |
| `ALTER TABLE ... RENAME COLUMN` | Add new → dual-write → migrate → drop old |
| `DROP COLUMN` on hot table | Remove from code first, drop in later release |

## Related Skills

- `database/knowledge-schema-design` — schema the migrations evolve
- `deployment/execution-cicd-pipeline` — migrations in CI/CD
- `reviewers/review-database` — migration review criteria
