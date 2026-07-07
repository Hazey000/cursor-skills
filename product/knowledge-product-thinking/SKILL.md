---
name: knowledge-product-thinking
description: >-
  Product thinking principles for engineers covering jobs-to-be-done, outcome
  over output, user empathy, and product-minded engineering. Use when making
  product trade-offs, prioritizing features, or ensuring engineering decisions
  serve user needs.
---

# Product Thinking

## Purpose

Enable engineers to make product-aware technical decisions.

## Core Mindset

**Outcome over output**: The value isn't the code shipped, it's the user problem solved.

## Jobs to Be Done (JTBD)

Users "hire" products to accomplish jobs:
- **Functional job**: "Help me track my expenses"
- **Emotional job**: "Make me feel in control of my finances"
- **Social job**: "Show my team I'm organized"

**Engineering implication**: Features should map to jobs. If a feature doesn't serve a job, question its priority.

## Product-Minded Engineering Questions

Before building, ask:
1. Who is the user and what problem are they solving?
2. How will we know this succeeded? (measurable outcome)
3. What's the simplest thing that could work?
4. What can we learn before building the full solution?
5. What happens if we don't build this?

## Decision Framework: Build vs Skip

| Signal | Build | Skip/Defer |
|--------|-------|------------|
| User requests | Many users, same job | One user, edge case |
| Impact | Unlocks a workflow | Marginal improvement |
| Reversibility | Easy to remove if wrong | Permanent commitment |
| Learning | Validates key assumption | No uncertainty to resolve |

## Engineering Trade-offs with Product Impact

| Engineering decision | Product impact |
|---------------------|----------------|
| Response time optimization | User perceived performance |
| Error message quality | User trust and self-service |
| API design | Developer experience (if API is the product) |
| Data model flexibility | Future feature velocity |
| Offline support | Usability in poor connectivity |

## Related Skills

- `product/knowledge-mvp-strategy` — scoping to minimum viable
- `product/knowledge-prioritization` — choosing what to build
- `poc/knowledge-poc-methodology` — validating before building
