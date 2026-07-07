---
name: knowledge-mvp-strategy
description: >-
  Minimum viable product scoping principles covering what to include, what to
  cut, and how to validate assumptions with minimal engineering effort. Use
  when scoping new features, defining project phases, or debating scope.
---

# MVP Strategy

## Purpose

Scope the smallest possible thing that validates the core assumption and delivers value.

## What MVP Actually Means

- **Minimum**: The least effort that still validates
- **Viable**: Actually usable, not a broken fragment
- **Product**: Solves a real problem for real users

MVP is NOT: a buggy version, a prototype with no UX consideration, or Phase 1 of a waterfall plan.

## Scoping Framework

### Step 1: Identify the Core Assumption
What must be true for this product to succeed? Build only what tests that.

### Step 2: Map Must-Have vs Nice-to-Have

| Category | Include in MVP | Defer |
|----------|---------------|-------|
| Core value proposition | Yes | — |
| Essential usability (login, error states) | Yes | — |
| Edge case handling | Only critical paths | Yes |
| Performance optimization | "Good enough" | Yes |
| Admin tools | Manual workarounds | Yes |
| Integrations | 1 (most common) | Yes |
| Customization | Hardcode sensible defaults | Yes |

### Step 3: Define "Done" for MVP
- Specific measurable outcome (users complete task, conversion rate, etc.)
- Timeline (timeboxed, typically 2-6 weeks)
- What feedback you'll collect

## Cutting Heuristics

Things to cut aggressively:
- Settings pages (hardcode good defaults)
- Email templates (plain text is fine)
- Admin dashboards (use database queries)
- Multiple user roles (start with one)
- Analytics (add after validating core value)
- Edge cases <5% of users will hit

## Anti-Patterns

- **MVP = everything but smaller**: Still a 6-month project, just with fewer features
- **MVP = crappy product**: No quality, no UX thought
- **MVP with no validation plan**: Ship and hope
- **Scope creep via "quick additions"**: Death by a thousand small features

## Related Skills

- `product/knowledge-product-thinking` — product mindset informing MVP
- `poc/knowledge-poc-methodology` — PoC before MVP (validate feasibility)
- `product/knowledge-prioritization` — ranking what goes in
