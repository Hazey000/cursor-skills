---
name: execution-poc-scaffold
description: >-
  Step-by-step procedure to bootstrap a new PoC project. Covers hypothesis
  definition, assumption ranking, minimal tech stack selection, project
  structure, implementation, and findings documentation. Includes README and
  repo templates. Use when starting a new PoC or helping a user set one up.
---

# PoC Scaffold

Bootstrap a PoC that stays focused on proving feasibility, not building a product.

## Procedure

### Step 1: Define the Hypothesis

Write a single sentence:

```
We believe [technical approach] can [achieve outcome] within [constraint].
```

If you can't write this sentence, you're not ready to start coding. Go back and clarify with stakeholders.

**Bad:** "We want to try Kafka."
**Good:** "We believe Kafka Streams can process our clickstream events with < 100ms p99 latency at 50K events/sec on 3 brokers."

### Step 2: Identify and Rank Assumptions

List every assumption that must hold for the hypothesis to be true. Rank by `risk = likelihood_of_failure × cost_of_being_wrong`.

The PoC targets the top 1–3 assumptions. Everything else is out of scope.

```markdown
| # | Assumption | Likelihood of Failure | Cost if Wrong | Priority |
|---|-----------|----------------------|---------------|----------|
| 1 | Library X supports streaming responses | Medium | High (blocker) | **PoC this** |
| 2 | Latency stays under budget at scale | High | High (blocker) | **PoC this** |
| 3 | Team can learn framework in time | Low | Medium | Skip |
```

### Step 3: Choose Minimal Tech Stack

Pick the **fastest path to answering the hypothesis**. Not the best tech — the fastest.

| Decision | Default to | Avoid |
|----------|-----------|-------|
| Language | Whatever the team knows best | New languages "to evaluate" |
| Framework | Minimal / none | Full-stack frameworks |
| Database | SQLite / in-memory / flat files | Managed cloud services |
| Infra | Local / single machine | Kubernetes, Terraform |
| Dependencies | As few as possible | "Let's also evaluate X" |

Exception: if the hypothesis is specifically about a technology choice, use that technology.

### Step 4: Set Up Project Structure

```
poc-[name]/
├── README.md              # Hypothesis, setup, results (see template below)
├── EVALUATION.md           # Findings and decision (fill in at end)
├── src/                    # Implementation — flat is fine
│   └── ...
├── data/                   # Sample data, fixtures, test inputs
│   └── ...
├── scripts/                # Run scripts, benchmarks, data generators
│   ├── run.sh
│   └── benchmark.sh
└── docs/                   # Screenshots, diagrams, notes (optional)
```

**What to include:**
- README with setup instructions (someone else must be able to run it)
- Scripts to reproduce the key experiment
- Sample data or data generation scripts
- Evaluation notes

**What to skip:**
- Authentication / authorisation
- CI/CD pipelines
- Monitoring / alerting
- Comprehensive error handling
- Tests (unless testing IS the hypothesis)
- Code review / linting config
- Docker / containerisation (unless infra is the hypothesis)

### Step 5: Implement the Core Proof

Write the minimum code to test the hypothesis. Guiding rules:

1. **Hardcode aggressively** — config files, env vars, and CLI args are overhead. Hardcode until it blocks you.
2. **Skip abstractions** — no interfaces, no dependency injection, no clean architecture. One file is fine.
3. **Fake the surroundings** — mock external services, use static data, stub what isn't being tested.
4. **Measure what matters** — if latency is the question, add timing. If accuracy is the question, add scoring. Nothing else.
5. **Stop when answered** — the moment you can fill in the EVALUATION.md, stop coding.

### Step 6: Document Findings

Fill in `EVALUATION.md` with:

```markdown
# Evaluation: [PoC Name]

## Hypothesis
[Copy from README]

## Approach
[What you built, key technical choices, and why]

## Results

### [Criterion 1: e.g. Latency]
- **Target:** < 200ms p95
- **Actual:** 145ms p95
- **Verdict:** ✅ Pass

### [Criterion 2: e.g. Accuracy]
- **Target:** > 90% F1
- **Actual:** 78% F1
- **Verdict:** ❌ Fail

## Raw Data
[Link to benchmarks, logs, screenshots]

## Decision
[Go / No-Go / Pivot]

## Rationale
[Why this decision, what evidence supports it]

## Recommendations
[Next steps — if Go: what to build properly. If No-Go: what to try instead.]

## Surprises / Learnings
[Anything unexpected that's worth recording for future reference]
```

## README Template

```markdown
# PoC: [Name]

## Hypothesis

We believe [approach] can [achieve outcome] within [constraint].

## Riskiest Assumptions

1. [Assumption being tested]
2. [Assumption being tested]

## Success Criteria

- [ ] [Measurable criterion 1]
- [ ] [Measurable criterion 2]

## Failure / Kill Criteria

- [ ] [Condition that means stop]

## Timebox

**Duration:** [X days]
**Start:** [date]
**End:** [date]

## Setup

```bash
# Prerequisites
# [list required tools]

# Run
# [commands to reproduce the experiment]
```

## Tech Stack

| Component | Choice | Rationale |
|-----------|--------|-----------|
| Language | [X] | [why] |
| [other] | [X] | [why] |

## Status

- [ ] Hypothesis defined
- [ ] Assumptions ranked
- [ ] Core proof implemented
- [ ] Evaluation complete
- [ ] Decision documented
```

## Timeboxing Guidance

| When | Action |
|------|--------|
| Start | Set a hard deadline. Put it in the README. Tell stakeholders. |
| 50% through | Check: are you testing the hypothesis or yak-shaving? |
| 75% through | You should be measuring, not coding. |
| Timebox expires | Stop. Write EVALUATION.md. Decide. |
| Want to extend | Requires explicit approval with written justification. Max 1 extension. |

## Anti-Patterns to Watch For

- Adding "just one more feature" after the hypothesis is answered
- Refactoring PoC code for readability
- Setting up dev tooling (linters, formatters, pre-commit hooks)
- Writing unit tests for throwaway code
- Deploying to a "real" environment
- Treating the PoC repo as the start of the real project

## Cross-References

- See `poc/knowledge-poc-methodology` — principles, decision frameworks, and when not to PoC
- See `poc/execution-evaluation-criteria` — how to define and score evaluation dimensions
- See `poc/execution-spike-design` — for shorter, narrower technical investigations
