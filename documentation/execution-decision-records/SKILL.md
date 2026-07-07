---
name: execution-decision-records
description: >-
  Write ADRs (Architecture Decision Records) and RFCs (Requests for Comments).
  Covers when to use each format, templates, and maintenance. Use when
  documenting technical decisions or proposing significant changes.
---

# Decision Records

## Purpose

Capture the WHY behind technical decisions so future team members understand context.

## ADR vs RFC

| Format | When | Audience | Lifecycle |
|--------|------|----------|-----------|
| ADR | Decision already made (or being made by small group) | Future team | Permanent record |
| RFC | Proposing change that needs broader input | Current team | Active discussion → decision |

## ADR Template (Nygard Format)

```markdown
# N. [Decision Title]

Date: YYYY-MM-DD

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-N]

## Context
[What problem are we solving? What constraints exist?]

## Decision
[What we decided to do. Start with "We will..."]

## Consequences
[Positive and negative trade-offs of this decision]
```

## RFC Template

```markdown
# RFC: [Title]

Author: [name]  |  Status: [Draft | Review | Accepted | Rejected]  |  Date: YYYY-MM-DD

## Summary
[One paragraph: what you're proposing]

## Motivation
[Why is this needed? What problem does it solve?]

## Proposal
[Detailed technical proposal]

## Alternatives Considered
[Other approaches and why they were rejected]

## Open Questions
[Unresolved decisions for discussion]
```

## When to Write a Decision Record

- Choosing between technologies
- Significant architectural change
- Convention that may seem arbitrary later
- Any decision you've debated for >30 minutes

## Related Skills

- `documentation/knowledge-technical-writing` — writing principles
- `architecture/knowledge-clean-architecture` — architectural decisions to record
