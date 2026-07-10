---
name: self-review
description: >-
  Run a self-review of uncommitted or branch changes against this repo's
  existing reviewer checklists (reviewers/review-*), routing changed files to
  the relevant reviewers and producing one consolidated findings report. Use
  when the user asks to self-review their changes, review a diff before
  opening a PR, or run the reviewer checklists against their code.
---

# Self-Review Against Reviewer Checklists

Route a diff to the relevant `reviewers/review-*` skills already in this repo, apply their checklists, and produce one consolidated report — so issues get caught before a human reviewer sees them.

## Core Principle

**Route, load, apply, consolidate.**

Don't re-derive review criteria — this repo already has detailed, severity-rated checklists in `reviewers/`. Figure out which ones apply to the changed files, load them, apply them to the actual diff content, and merge the results into a single report.

## Requirements

This skill reads sibling skill files at review time, so it only works when the following are present relative to this skill's location (they already are, in this repo):

- `reviewers/review-api/SKILL.md`
- `reviewers/review-security/SKILL.md`
- `reviewers/review-database/SKILL.md`
- `reviewers/review-accessibility/SKILL.md`
- `reviewers/review-ux/SKILL.md`
- `reviewers/review-performance/SKILL.md`
- `reviewers/review-architecture/SKILL.md`
- `reviewers/review-production-readiness/SKILL.md`

If copying only `self-review/SKILL.md` into another project, copy the `reviewers/` directory alongside it, or this skill has nothing to load.

## Workflow

### Step 1: Determine Diff Scope

```
AskQuestion:
  title: "Self-Review Scope"
  questions:
    - id: scope
      prompt: "What should I review?"
      options:
        - "Uncommitted changes (Recommended)"
        - "Current branch vs main/master"
        - "Specific files"
```

If "Specific files", ask which paths.

### Step 2: Get the Changed Files and Diff

Via `Shell` (read-only):

```bash
git diff --name-status                 # uncommitted
git diff                                # uncommitted, full diff
```

or, for branch vs base:

```bash
git diff <base>...HEAD --name-status
git diff <base>...HEAD
```

### Step 3: Route Files to Reviewers

Apply this mapping to the changed file list. A file can match multiple rows — include every matching reviewer.

| Signal (path or content pattern) | Reviewer to load |
|---|---|
| `**/api/**`, `**/routes/**`, `**/controllers/**`, `*.proto`, OpenAPI/GraphQL schema files | `reviewers/review-api` |
| `**/auth/**`, `**/middleware/**`, files touching secrets, env vars, user input parsing, crypto | `reviewers/review-security` |
| `**/migrations/**`, `*.sql`, schema/model definition files | `reviewers/review-database` |
| `*.tsx`, `*.jsx`, `*.vue`, `**/components/**` | `reviewers/review-accessibility`, `reviewers/review-ux` |
| Hot-path code, query loops, new caching layers, N+1-prone code | `reviewers/review-performance` |
| New services, `Dockerfile`, `**/infra/**`, deploy/CI config, significant structural changes | `reviewers/review-architecture`, `reviewers/review-production-readiness` |

If nothing matches any row, still run a lightweight pass with `reviewers/review-architecture` as a structural sanity fallback — never skip review entirely.

Present the routing decision briefly before proceeding, e.g.:

```
Routing 6 changed files -> review-api, review-security (auth middleware touched), review-database (1 migration)
```

### Step 4: Load Each Applicable Checklist

For every reviewer selected in Step 3, `Read` its `SKILL.md` file in full to pull its checklist tables, severity levels, and output format. Do this for all selected reviewers before evaluating — don't interleave loading and evaluating.

### Step 5: Apply Checklists to the Diff

For each check in each loaded checklist:
- Use the diff to identify candidate locations.
- `Read` the full changed file (not just the diff hunk) when context beyond the hunk is needed to judge a check correctly (e.g. checking whether an endpoint enforces auth requires seeing the surrounding route setup, not just the changed lines).
- Only report a finding if the check is actually violated by this diff — do not flag pre-existing issues in untouched code unless they are directly relevant to the change (e.g. a new caller of an already-insecure function).

### Step 6: Produce One Consolidated Report

Reuse the severity vocabulary (CRITICAL / WARNING / SUGGESTION) and per-finding format already used by the loaded reviewers. Merge everything into a single report:

```markdown
## Self-Review: [branch name or "uncommitted changes"]

**Scope**: [files/diff reviewed]
**Reviewers applied**: [list, with one-line reason each]
**Overall**: [PASS | PASS WITH WARNINGS | FAIL]

### Findings by Reviewer

#### review-security
##### [CRITICAL] Missing auth check on new endpoint
**Location**: `src/api/users.ts:42`
**Issue**: [...]
**Fix**: [...]

#### review-database
##### [WARNING] Migration missing a down/rollback step
...

### Summary
| Reviewer | Critical | Warning | Suggestion |
|---|---|---|---|
| review-security | 1 | 0 | 0 |
| review-database | 0 | 1 | 0 |
| **Total** | **1** | **1** | **0** |

### Fix Before Opening PR
1. [All CRITICALs, plus any high-value WARNINGs, in priority order]
```

If a reviewer's checklist produced zero findings, still list it in "Reviewers applied" with a "no issues found" note — this shows the coverage, not just the problems.

### Step 7: Offer Next Step

```
AskQuestion:
  title: "Self-Review Complete"
  questions:
    - id: next
      prompt: "Found N issue(s) (X critical, Y warning). What next?"
      options:
        - "Just the report is fine (Recommended)"
        - "Fix the straightforward ones for me"
        - "Walk me through each finding one by one"
```

If "Fix the straightforward ones for me", only apply fixes that are unambiguous and low-risk (e.g. adding a missing rollback step, parameterizing a query) — leave anything requiring a design decision for the user, and list what was and wasn't auto-fixed.

## Handling Edge Cases

- **No changes to review**: tell the user rather than producing an empty report.
- **Diff too large for careful review**: warn the user and offer to review file-by-file or module-by-module instead of everything at once.
- **A matched reviewer file is missing**: skip it, note the gap in the report ("review-performance skipped — reviewers/review-performance/SKILL.md not found"), and continue with the others.

## Important Notes

- This complements, not replaces, human code review — say so in the report if the user seems to be treating it as a substitute for peer review.
- Findings should cite the exact file and line where possible, mirroring the loaded reviewers' own output format.
- Keep the routing table in Step 3 in sync if new `reviewers/review-*` skills are added to this repo later.
