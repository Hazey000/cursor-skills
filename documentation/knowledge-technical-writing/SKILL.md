---
name: knowledge-technical-writing
description: >-
  Technical writing principles for engineers covering clarity, audience
  awareness, document structure, and progressive disclosure. Use when writing
  documentation, README files, technical guides, or reviewing documentation
  quality.
---

# Technical Writing

## Purpose

Write documentation that engineers actually read and find useful.

## Core Principles

1. **Know your audience**: Expertise level determines depth and terminology
2. **Lead with the answer**: Don't bury the useful information
3. **Progressive disclosure**: Overview first, details on demand
4. **Concrete over abstract**: Examples > explanations
5. **Maintain ruthlessly**: Outdated docs are worse than no docs

## Document Types and Their Purpose

| Type | Audience | Lifespan | Key quality |
|------|----------|----------|-------------|
| README | New contributor | Long | Gets someone running in <5 minutes |
| API reference | Consumer developer | Long | Complete, accurate, searchable |
| Tutorial | Learner | Medium | Achievable end-to-end walkthrough |
| How-to guide | Practitioner | Medium | Solves a specific problem |
| ADR | Future team | Permanent | Captures WHY, not just WHAT |
| Runbook | On-call engineer | Long | Step-by-step under pressure |

## Writing Heuristics

- **One idea per paragraph**
- **Use headers liberally** (scannable > readable)
- **Tables over prose** for comparisons and options
- **Code examples that actually work** (tested, copy-pasteable)
- **Link, don't repeat** (DRY for docs)

## Anti-Patterns

- Wall of text with no headers
- Outdated information (worse than no docs)
- "Obvious" comments that explain what, not why
- Documentation that requires reading other documentation first
- Generated docs with no human-written context

## Related Skills

- `documentation/execution-api-docs` — API documentation specifically
- `documentation/execution-runbooks` — operational documentation
- `documentation/execution-decision-records` — ADRs and RFCs
