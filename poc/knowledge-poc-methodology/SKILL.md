---
name: knowledge-poc-methodology
description: >-
  PoC philosophy and decision framework: proving feasibility of riskiest
  assumptions, not building a mini-production system. Covers PoC vs prototype
  vs MVP vs pilot, success/failure criteria, risk-driven development, and when
  NOT to build a PoC. Use when planning, scoping, or evaluating a PoC.
---

# PoC Methodology

A Proof of Concept answers one question: **can this work?** Everything else is noise.

## Core Principle

> A PoC proves feasibility of the riskiest assumption in the cheapest possible way.

It is not a demo, not a prototype, and not a "v0.1". It is a disposable experiment designed to produce a **yes/no decision** with evidence.

## PoC vs Prototype vs MVP vs Pilot

| | PoC | Prototype | MVP | Pilot |
|---|---|---|---|---|
| **Purpose** | Prove feasibility | Explore UX/design | Validate market fit | Validate in production |
| **Audience** | Engineers / decision-makers | Users / stakeholders | Real customers | Limited real users |
| **Fidelity** | Minimal — ugly is fine | Medium — looks real | Functional — shippable | Production — full ops |
| **Lifespan** | Days to 2 weeks | Weeks | Months | Months |
| **Outcome** | Go/no-go decision | Design direction | Product-market signal | Operational confidence |
| **Code destiny** | Throw away | Mostly throw away | Evolve into product | Evolve or replace |

**Key distinction:** A PoC that "works" does NOT become the product. If you're tempted to ship the PoC, you're building a prototype.

## Risk-Driven Development

PoCs exist to de-risk. Prioritise by:

1. **Identify all assumptions** — list what must be true for the project to succeed
2. **Rank by risk** — what is most likely to be wrong or hardest to achieve?
3. **Rank by cost of being wrong** — what kills the project if the assumption fails?
4. **PoC the intersection** — high probability of failure × high cost of failure

```
Risk Priority = Likelihood_of_failure × Cost_of_being_wrong

PoC targets the TOP item on this list, not the most interesting one.
```

### Common Riskiest Assumptions

- "The API can handle our throughput requirements"
- "The model is accurate enough for this use case"
- "We can integrate with system X within our constraints"
- "This algorithm runs within our latency budget"
- "The data is clean enough / available enough"

## Timeboxing

| PoC Scope | Timebox |
|---|---|
| API integration feasibility | 1–2 days |
| Algorithm/model evaluation | 2–5 days |
| Architecture pattern validation | 3–5 days |
| Cross-system integration | 1–2 weeks |
| Hardware/infrastructure | 1–2 weeks |

**Hard rule:** When the timebox expires, you stop and evaluate. Extensions require an explicit decision with justification — never silently drift.

## Success and Failure Criteria

Define these **before writing code**. A PoC without criteria is just tinkering.

### Structure

```
Hypothesis: [What we believe to be true]
Success criteria: [Measurable conditions that confirm the hypothesis]
Failure criteria: [Conditions that disprove it — be specific]
Kill criteria: [Conditions that mean stop immediately, don't finish the timebox]
```

### Examples

```
Hypothesis: PostgreSQL full-text search handles our query patterns at scale.
Success: p95 query latency < 200ms at 10K concurrent queries on 50M rows.
Failure: p95 > 500ms, or requires schema that breaks existing access patterns.
Kill: p95 > 2s at any scale — fundamentally wrong tool.
```

### Good Criteria Are

- **Measurable** — numbers, not feelings ("fast enough" is not a criterion)
- **Binary** — pass/fail, not a spectrum
- **Defined upfront** — moving goalposts invalidate the experiment
- **Honest** — include conditions where you admit it failed

## When NOT to Build a PoC

- **The answer is already known** — someone on the team (or the internet) has done this before. Search first.
- **The risk is purely schedule risk** — a PoC won't tell you if you have enough time, only if something is technically feasible.
- **You're avoiding a decision** — "let's PoC it" is sometimes code for "I don't want to commit". If the options are well-understood, decide.
- **The riskiest part is organisational, not technical** — no amount of code proves stakeholder buy-in.
- **You need a prototype** — if the question is "will users like this?", you need UX testing, not a PoC.
- **The cost of the PoC ≈ the cost of just building it** — if the feature is small enough, skip the experiment and build it properly.

## Decision Framework

```
Is the riskiest assumption technical?
├─ No → Don't PoC. Address the actual risk (org, UX, market).
└─ Yes
   ├─ Has someone already proven this? → Don't PoC. Use their evidence.
   └─ No
      ├─ Can you prove/disprove in ≤ 2 weeks? → PoC it.
      └─ No → Break into smaller assumptions, PoC the riskiest sub-part.
```

## Documenting a PoC

Every PoC must produce a written artefact, even if it fails — especially if it fails. Minimum:

1. **Hypothesis** — what were you testing
2. **Approach** — what you built and why that approach
3. **Results** — data, measurements, observations
4. **Decision** — go / no-go / pivot, and why
5. **Recommendations** — what to do next

Failure is a valid, valuable outcome. Document it so nobody repeats the experiment.

## Anti-Patterns

- **PoC creep** — "let me just add auth… and logging… and tests…" — you're building a product
- **Success theatre** — cherry-picking results to justify a predetermined decision
- **Zombie PoC** — PoC that "sort of works" but never gets a clear verdict, lingers in limbo
- **Framework tourism** — using the PoC as an excuse to try a new tech stack rather than test the hypothesis
- **Ignoring negative results** — the PoC failed but you build anyway because of sunk cost

## Cross-References

- See `poc/execution-poc-scaffold` — step-by-step procedure to bootstrap a PoC
- See `poc/execution-spike-design` — for shorter, more focused technical investigations
- See `poc/execution-evaluation-criteria` — for defining and scoring evaluation dimensions
