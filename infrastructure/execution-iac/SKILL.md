---
name: execution-iac
description: >-
  Infrastructure as Code workflows covering state management, module design,
  environment promotion, and drift detection. Tool-agnostic principles for
  managing infrastructure declaratively. Use when setting up IaC, designing
  module structure, or reviewing infrastructure code.
---

# Infrastructure as Code

## Purpose

Step-by-step guidance for managing infrastructure declaratively.

## Workflow

### Step 1: Structure the Codebase

```
infra/
  modules/          # Reusable, parameterized components
    networking/
    compute/
    database/
  environments/     # Environment-specific configurations
    dev/
    staging/
    production/
  global/           # Shared resources (DNS, IAM)
```

### Step 2: Module Design Principles

- One module = one logical resource group
- Inputs via variables with sensible defaults
- Outputs for values other modules need
- No hardcoded environment-specific values
- Version modules (Git tags or registry)

### Step 3: State Management

- Remote state storage (encrypted, versioned)
- State locking (prevent concurrent modifications)
- Separate state per environment
- Never edit state manually

### Step 4: Environment Promotion

- Same modules, different variables per environment
- Promote by applying same version to next environment
- Dev → Staging → Production pipeline
- Feature branches for infrastructure changes (plan in PR)

### Step 5: Safety Practices

- Plan before apply (always review the diff)
- Automated plan in PR (CI comments with plan output)
- Apply only from CI (not local machines)
- Drift detection (scheduled plan with alerting)

## Checklist

- [ ] Remote state with locking
- [ ] Modules versioned and reusable
- [ ] No secrets in IaC code (reference secrets manager)
- [ ] CI runs plan on PR, apply on merge
- [ ] Environments use same modules with different vars
- [ ] Drift detection scheduled

## Related Skills

- `infrastructure/knowledge-cloud-architecture` — what to provision
- `deployment/execution-cicd-pipeline` — IaC in CI/CD
- `security/knowledge-secure-by-design` — infrastructure security
