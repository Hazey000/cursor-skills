---
name: knowledge-data-modeling
description: >-
  Data modeling from conceptual to physical level. Covers entity relationship
  modeling, cardinality, inheritance strategies, and translating domain models
  to schemas. Use when starting new database designs or mapping domain concepts
  to tables.
---

# Data Modeling

## Purpose

Systematic approach to translating domain concepts into database structures.

## Modeling Levels

| Level | Audience | Contains | Tools |
|-------|----------|----------|-------|
| Conceptual | Business stakeholders | Entities, relationships | Whiteboard, ER diagrams |
| Logical | Developers, architects | Attributes, keys, cardinality | ER diagrams, UML |
| Physical | DBAs, developers | Tables, columns, types, indexes | DDL, migration files |

## Relationship Patterns

| Cardinality | Implementation |
|-------------|---------------|
| 1:1 | Foreign key with UNIQUE constraint, or same table |
| 1:N | Foreign key on the "many" side |
| M:N | Junction/join table with composite key |
| Self-referential | Foreign key referencing same table |

## Inheritance Strategies

| Strategy | When to use | Trade-off |
|----------|-------------|-----------|
| Single table | Few subtypes, similar attributes | Sparse nulls |
| Table-per-type | Many attributes per subtype | Joins needed |
| Table-per-concrete | Subtypes rarely queried together | No shared table |

## Modeling Heuristics

- Nouns → entities/tables
- Verbs → relationships or junction tables
- Adjectives → attributes or enum columns
- "Has many" → 1:N foreign key
- "Belongs to many" → M:N junction table

## Related Skills

- `database/knowledge-schema-design` — physical schema from the model
- `architecture/knowledge-ddd` — domain model driving data model
