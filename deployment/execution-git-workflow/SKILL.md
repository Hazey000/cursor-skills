---
name: execution-git-workflow
description: >-
  Git branching strategy, PR workflow, commit conventions, and release process.
  Decision framework for choosing a workflow. Use when establishing team Git
  conventions, designing branch protection rules, or reviewing Git practices.
---

# Git Workflow

## Purpose

Choose and implement a Git workflow appropriate for team size and release cadence.

## Workflow Selection

| Workflow | Best for | Release cadence |
|----------|----------|-----------------|
| Trunk-based | Small teams, CD, feature flags | Continuous |
| GitHub Flow | Most teams, simple branching | On merge to main |
| Git Flow | Strict release schedules, multiple versions | Scheduled releases |

**Default recommendation**: Trunk-based with short-lived branches (< 2 days).

## Branch Convention

- `main` — always deployable, protected
- `feature/description` — short-lived work branches
- `fix/description` — bug fix branches
- `release/vX.Y.Z` — only if using Git Flow

## Commit Message Convention

Format: `type(scope): description`

Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

Examples:
- `feat(auth): add OAuth2 login flow`
- `fix(api): handle null response from payment provider`
- `refactor(orders): extract validation to dedicated service`

## Pull Request Workflow

1. Create branch from main (keep it short-lived)
2. Push early and often (enables CI feedback)
3. Open PR when ready for review (or draft PR for early feedback)
4. CI must pass before review
5. At least one approval required
6. Squash merge to main (clean history)
7. Delete branch after merge

## Branch Protection Rules

- [ ] Require CI to pass before merge
- [ ] Require at least 1 approval
- [ ] Dismiss stale approvals on new push
- [ ] No direct push to main
- [ ] Linear history (squash or rebase merges)

## Related Skills

- `deployment/execution-cicd-pipeline` — pipeline triggered by Git events
- `deployment/knowledge-cicd-principles` — CI requires frequent integration
