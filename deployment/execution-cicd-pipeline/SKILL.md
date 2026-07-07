---
name: execution-cicd-pipeline
description: >-
  Design a CI/CD pipeline from scratch. Step-by-step procedure covering stage
  design, artifact management, environment promotion, and failure handling.
  Use when setting up CI/CD for new projects or improving existing pipelines.
---

# CI/CD Pipeline Design

## Purpose

Step-by-step procedure for designing an effective CI/CD pipeline.

## Workflow

### Step 1: Define Pipeline Stages

Minimum viable pipeline:
1. **Check** (seconds): lint, format, type check
2. **Test** (minutes): unit tests, coverage threshold
3. **Build** (minutes): compile, package, create artifact
4. **Deploy staging** (minutes): deploy to pre-production
5. **Verify** (minutes): smoke tests against staging
6. **Deploy production**: release to users

### Step 2: Optimize for Speed

- Parallelize independent steps (lint ‖ test ‖ security scan)
- Cache dependencies between runs
- Only rebuild/retest what changed (affected tests, monorepo filters)
- Fast-fail: cheapest checks first

### Step 3: Artifact Management

- Build artifact ONCE, deploy the SAME artifact everywhere
- Tag artifacts with commit SHA
- Store in artifact registry (container registry, package registry)
- Promote artifact between environments (don't rebuild)

### Step 4: Environment Promotion

```
PR → [check + test + build] → merge to main → [deploy staging] → [verify] → [deploy prod]
```

- Staging deployment automatic on merge
- Production deployment: automatic (CD) or gated (approval)
- Same configuration shape across environments (different values via env vars)

### Step 5: Failure Handling

- Flaky test quarantine (track and isolate, don't block pipeline)
- Automatic rollback on health check failure post-deploy
- Notifications to PR author on failure
- Pipeline run logs accessible and searchable

## Checklist

- [ ] All checks pass before merge allowed
- [ ] Artifact built once, promoted through environments
- [ ] Dependencies cached for speed
- [ ] Independent stages parallelized
- [ ] Deployment to staging automatic on merge
- [ ] Smoke/health tests verify each deployment
- [ ] Rollback mechanism defined and tested
- [ ] Secrets injected at runtime, never in pipeline config
- [ ] Pipeline completes in <10 minutes for feedback loop

## Related Skills

- `deployment/knowledge-cicd-principles` — philosophy behind the pipeline
- `deployment/execution-git-workflow` — branch strategy feeding the pipeline
- `infrastructure/execution-containerization` — building container artifacts
