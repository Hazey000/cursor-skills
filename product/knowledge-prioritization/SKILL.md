---
name: knowledge-prioritization
description: >-
  Prioritization frameworks including RICE, ICE, MoSCoW, and opportunity cost
  thinking. Decision framework for choosing between competing priorities. Use
  when deciding what to build next, triaging backlogs, or justifying priority
  decisions.
---

# Prioritization

## Purpose

Systematic approaches to choosing what to build when you can't build everything.

## Frameworks

### RICE Score

| Factor | Meaning | Scale |
|--------|---------|-------|
| Reach | How many users/period | Estimated count |
| Impact | Effect per user | 3=massive, 2=high, 1=medium, 0.5=low, 0.25=minimal |
| Confidence | How sure are you | 100%=high, 80%=medium, 50%=low |
| Effort | Person-months | Estimated effort |

**Score = (Reach × Impact × Confidence) / Effort**

### ICE Score (Simpler)

| Factor | Scale |
|--------|-------|
| Impact | 1-10 |
| Confidence | 1-10 |
| Ease | 1-10 |

**Score = Impact × Confidence × Ease**

### MoSCoW (for fixed scope/time)

- **Must have**: System doesn't work without it
- **Should have**: Important but not critical
- **Could have**: Nice to have, include if time allows
- **Won't have (this time)**: Explicitly deferred

## Decision Heuristics

| Situation | Heuristic |
|-----------|-----------|
| Two features, similar value | Build the faster one first (learn sooner) |
| High uncertainty | Build the smallest experiment that resolves it |
| Blocking dependency | Unblock first, even if lower individual value |
| Quick win available | Do it (momentum + credibility) |
| Big bet vs many small | Usually: small wins accumulate faster |

## Opportunity Cost Thinking

Every "yes" is a "no" to something else. Ask:
- What could the team build instead with the same effort?
- What's the cost of delay for each option?
- Is there a time-sensitive window for any option?

## Related Skills

- `product/knowledge-product-thinking` — what drives priority
- `product/knowledge-mvp-strategy` — scoping the chosen priority
- `poc/execution-evaluation-criteria` — scoring frameworks for tech choices
