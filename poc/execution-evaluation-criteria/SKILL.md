---
name: execution-evaluation-criteria
description: >-
  How to define and score evaluation criteria for technology choices and PoC
  outcomes. Covers dimension identification, weighting, scoring rubrics,
  decision matrices, and common bias traps. Includes a reusable evaluation
  matrix template. Use when comparing technologies, scoring PoC results, or
  making build-vs-buy decisions.
---

# Evaluation Criteria

Structured evaluation turns subjective opinions into defensible decisions. This procedure works for technology selection, PoC outcomes, vendor comparison, and build-vs-buy decisions.

## Procedure

### Step 1: Identify Evaluation Dimensions

Start with dimensions relevant to the decision. Don't use all of these — pick 5–8 that actually matter.

| Category | Dimensions |
|----------|-----------|
| **Technical** | Performance, scalability, reliability, security, latency |
| **Developer Experience** | Learning curve, documentation quality, debugging tools, API ergonomics |
| **Ecosystem** | Community size, plugin/extension availability, third-party integrations |
| **Maturity** | Production track record, release cadence, backward compatibility |
| **Operational** | Deployment complexity, monitoring support, upgrade path, disaster recovery |
| **Cost** | Licensing, infrastructure, engineering time, ongoing maintenance |
| **Strategic** | Vendor lock-in risk, alignment with existing stack, hiring market |

**Rule:** If a dimension doesn't differentiate between options, drop it. If all candidates score the same on "security", it's not helping the decision.

### Step 2: Weight Dimensions by Project Priority

Assign weights that sum to 100. This forces trade-off conversations.

```
Performance:     30  ← This project is latency-sensitive
Cost:            25  ← Budget-constrained
DX:              20  ← Small team, velocity matters
Maturity:        15  ← Production deployment in 3 months
Ecosystem:       10  ← Few integrations needed
                ───
Total:          100
```

**How to set weights:**
1. Each stakeholder ranks dimensions independently
2. Discuss disagreements — they reveal misaligned priorities
3. Converge on weights before scoring (prevents gaming)

**If you can't agree on weights, that's the real problem.** Resolve priority alignment before evaluating options.

### Step 3: Define a Scoring Rubric

Use a consistent scale. Define what each score means for each dimension — abstract scores like "4/5" without criteria are meaningless.

#### Recommended: 1–5 Scale with Anchors

```markdown
| Score | Meaning |
|-------|---------|
| 5 | Exceptional — exceeds requirements, clear advantage |
| 4 | Strong — meets all requirements with minor advantages |
| 3 | Adequate — meets core requirements, no major concerns |
| 2 | Weak — partially meets requirements, notable gaps |
| 1 | Unacceptable — fails requirements or introduces serious risk |
```

#### Per-Dimension Anchors (Example)

```markdown
### Performance (weight: 30)
| Score | Definition |
|-------|-----------|
| 5 | < 50ms p99, linear scaling to 10x current load |
| 4 | < 100ms p99, scaling with minor tuning |
| 3 | < 200ms p99 (our target), scaling unclear |
| 2 | 200–500ms p99, scaling requires significant effort |
| 1 | > 500ms p99 or cannot scale |
```

Anchors prevent score inflation and make different evaluators' scores comparable.

### Step 4: Evaluate Candidates

Score each candidate against each dimension. Rules:

1. **Evidence-based** — every score must cite a source (benchmark, docs, spike result, production data)
2. **One scorer per dimension** — or average independent scores (never discuss before scoring)
3. **Score against the rubric, not against other candidates** — relative scoring introduces anchoring bias
4. **Record the rationale** — the score without reasoning is useless in 6 months

### Step 5: Build the Decision Matrix

Calculate weighted scores and present.

```markdown
| Dimension | Weight | Option A | Score | Option B | Score | Option C | Score |
|-----------|--------|----------|-------|----------|-------|----------|-------|
| Performance | 30 | 145ms p99 | 4 | 95ms p99 | 4 | 310ms p99 | 2 |
| Cost | 25 | $0 (OSS) | 5 | $2K/mo | 3 | $500/mo | 4 |
| DX | 20 | Good docs, OK API | 4 | Sparse docs | 2 | Great DX | 5 |
| Maturity | 15 | 8yr, stable | 5 | 2yr, fast churn | 2 | 5yr, stable | 4 |
| Ecosystem | 10 | Large | 4 | Small | 2 | Medium | 3 |
| **Weighted** | | | **4.25** | | **2.85** | | **3.55** |

Weighted = Σ (weight/100 × score)
```

### Step 6: Make the Decision

The highest score is a **recommendation**, not an automatic decision. Check:

1. **Does any candidate have a score of 1 on a weight ≥ 20 dimension?** — That's likely a dealbreaker regardless of total score.
2. **Are the top two within 0.5 of each other?** — The data doesn't clearly differentiate them. Use secondary factors (team preference, strategic direction) as tiebreakers.
3. **Is the winner surprising?** — Re-examine weights and scores for bias. Surprise is fine if the evidence supports it.

Write the decision as:

```markdown
## Decision

**Selected:** [Option]
**Score:** [weighted score]
**Runner-up:** [Option] ([score])

**Rationale:** [2–3 sentences explaining why, referencing key differentiating dimensions]

**Risks accepted:** [What are you giving up by not choosing the runner-up?]

**Revisit if:** [Conditions that should trigger re-evaluation]
```

## Evaluation Matrix Template

```markdown
# Evaluation: [What are we deciding?]

**Date:** [YYYY-MM-DD]
**Evaluators:** [names]
**Options:** [list candidates]

## Context
[1–2 paragraphs: what problem we're solving, constraints, timeline]

## Dimensions and Weights

| # | Dimension | Weight | Rationale for Weight |
|---|-----------|--------|---------------------|
| 1 | [dim] | [0–100] | [why this weight] |
| 2 | [dim] | [0–100] | [why this weight] |
| ... | ... | ... | ... |
| | **Total** | **100** | |

## Scoring Rubric

### [Dimension 1] (weight: [N])
| Score | Definition |
|-------|-----------|
| 5 | [anchor] |
| 3 | [anchor] |
| 1 | [anchor] |

[Repeat for each dimension]

## Scores

| Dimension | Weight | [Option A] | | [Option B] | | [Option C] | |
|-----------|--------|-------|-|-------|-|-------|-|
| | | Evidence | Score | Evidence | Score | Evidence | Score |
| [dim 1] | [w] | [data] | [1-5] | [data] | [1-5] | [data] | [1-5] |
| [dim 2] | [w] | [data] | [1-5] | [data] | [1-5] | [data] | [1-5] |
| **Weighted Total** | | | **[X.XX]** | | **[X.XX]** | | **[X.XX]** |

## Decision

**Selected:** [Option]
**Runner-up:** [Option]

**Rationale:** [Why]

**Risks accepted:** [What we're giving up]

**Revisit if:** [Trigger conditions for re-evaluation]
```

## Common Bias Traps

| Bias | Description | Mitigation |
|------|-------------|-----------|
| **Anchoring** | First option evaluated sets the baseline, others scored relative to it | Score against rubric, not other options. Randomise evaluation order. |
| **Familiarity bias** | Favouring what the team already knows | Weight "learning curve" explicitly. Acknowledge it, don't hide it. |
| **Halo effect** | One impressive feature inflates all scores | Score each dimension independently. Don't discuss total until all dimensions scored. |
| **Sunk cost** | Favouring the option you've already invested time in | Evaluate as if starting from zero. Have someone with no prior investment score. |
| **Confirmation bias** | Seeking evidence that supports your preferred option | Assign someone to argue for each option's weaknesses (devil's advocate). |
| **Novelty bias** | Favouring shiny new technology over boring proven solutions | Include "maturity" and "production track record" as weighted dimensions. |
| **Authority bias** | Adopting a technology because a respected company uses it | Require evidence from your context, not theirs. "Netflix uses it" is not a score. |

## Tips

- **5–8 dimensions** is the sweet spot. Fewer than 5 is too coarse. More than 8 and weights become meaninglessly small.
- **Never change weights after scoring.** If the result surprises you, the weights were wrong — acknowledge it, re-weight, and re-score.
- **Time-bound the evaluation.** Set a deadline for the decision. Paralysis by analysis is a real risk.
- **Version the evaluation.** Decisions get revisited. Keep the matrix so future you understands why.

## Cross-References

- See `poc/knowledge-poc-methodology` — for understanding when evaluation follows a PoC
- See `poc/execution-poc-scaffold` — use evaluation criteria in the EVALUATION.md output
- See `poc/execution-spike-design` — for running parallel spikes before evaluation
