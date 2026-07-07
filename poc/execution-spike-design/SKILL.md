---
name: execution-spike-design
description: >-
  How to design and execute a technical spike. Covers framing the question,
  defining success criteria, timeboxing, choosing the narrowest experiment,
  documenting findings, and making a recommendation. Includes spike output
  template. Use when investigating unknowns, evaluating approaches, or
  answering technical questions with code.
---

# Spike Design

A spike is a short, focused investigation that answers a specific technical question with working code.

## Spike vs PoC

| | Spike | PoC |
|---|---|---|
| **Scope** | Single question | Feasibility of a concept |
| **Duration** | Hours to 3 days | Days to 2 weeks |
| **Output** | Answer + recommendation | Go/no-go decision + evidence |
| **Code** | Minimal, often a script | Small working system |
| **Formality** | Low — notes + code | Medium — README, evaluation doc |
| **When** | "Can we do X?" or "How does Y work?" | "Is this approach viable for our system?" |

Use a spike when the question is narrow. Use a PoC when you need to prove an integrated concept works.

## Procedure

### Step 1: Frame the Question

Write the spike as a single question:

```
How does [technology/approach] handle [specific scenario]?
Can we [achieve specific outcome] using [specific tool/technique]?
What is the [performance/behaviour] of [X] under [conditions]?
```

**Bad:** "Investigate caching options."
**Good:** "Can Redis Cluster handle our session store at 100K concurrent connections with < 5ms p99 read latency?"

One question per spike. If you have multiple questions, run multiple spikes.

### Step 2: Define Success Criteria

Before writing code, write down what "answered" looks like:

```
This spike is complete when:
- [ ] [Specific measurable outcome or observation]
- [ ] [Recommendation is written]
```

### Step 3: Timebox

| Spike Type | Timebox |
|---|---|
| API/library behaviour investigation | 2–4 hours |
| Performance characterisation | 4–8 hours |
| Integration feasibility | 1–2 days |
| Algorithm comparison | 1–3 days |

Set a hard stop. If you haven't answered the question by the timebox, the answer is either "it's more complex than expected" (which is itself a finding) or the question needs to be narrowed.

### Step 4: Choose the Narrowest Experiment

Strip everything that isn't directly testing the question:

- **Isolate the variable** — test one thing at a time
- **Use the simplest possible harness** — a script, a REPL, a notebook
- **Hardcode everything** — no config, no CLI args, no env vars
- **Use synthetic data** — don't plumb real data pipelines
- **Skip error handling** — let it crash, you're watching

### Step 5: Implement and Observe

Write code, run it, measure. Take notes as you go — timestamps, observations, surprising behaviours, error messages.

Capture:
- **Quantitative data** — latency, throughput, memory, accuracy
- **Qualitative observations** — DX friction, documentation quality, gotchas
- **Blockers encountered** — things that took longer than expected and why

### Step 6: Document and Recommend

Fill in the spike output template. The recommendation must be actionable — "we should do X" or "we should not do X because Y".

## Spike Output Template

```markdown
# Spike: [Question]

**Date:** [YYYY-MM-DD]
**Duration:** [actual time spent]
**Author:** [name]

## Question
[The specific question this spike answers]

## Approach
[What you did — tools, code, methodology. Keep brief.]

## Findings

### Key Result
[The direct answer to the question, 1–3 sentences]

### Data
[Measurements, benchmarks, observations — whatever evidence supports the answer]

| Metric | Value | Notes |
|--------|-------|-------|
| [metric] | [value] | [context] |

### Surprises / Gotchas
- [Unexpected finding 1]
- [Unexpected finding 2]

## Recommendation
[Clear, actionable recommendation: what to do next and why]

## Code
[Link to spike code, or inline if < 50 lines. Delete after decision is made.]
```

## When to Extend vs Kill

### Extend (max once, requires justification)

- You're close to an answer but hit a setup issue (not a fundamental blocker)
- The findings so far reveal a more precise question worth 1 more day
- Stakeholder requests a specific additional data point

### Kill

- The timebox expired with no clear direction — the question is too broad
- You found a definitive blocker — the answer is "no", stop here
- The question became irrelevant due to new information
- You're yak-shaving: more time setting up infrastructure than answering the question

### Neither — Escalate to PoC

- The spike revealed the question is bigger than expected and needs a proper PoC
- Multiple interacting assumptions need testing together
- The answer is "maybe, if several things work together"

## Running Multiple Spikes

When evaluating alternatives, run parallel spikes:

1. **Same question, different approaches** — e.g. "Can we do X with Redis?" and "Can we do X with DynamoDB?"
2. **Same timebox for each** — fair comparison
3. **Same success criteria** — apples to apples
4. **Present results together** — use `poc/execution-evaluation-criteria` for the comparison matrix

## Cross-References

- See `poc/knowledge-poc-methodology` — when a spike isn't enough and you need a full PoC
- See `poc/execution-evaluation-criteria` — for comparing multiple spike results systematically
- See `poc/execution-poc-scaffold` — when the spike reveals you need a full PoC
