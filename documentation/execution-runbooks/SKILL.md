---
name: execution-runbooks
description: >-
  Create operational runbooks for incident response. Covers runbook structure,
  diagnostic steps, remediation procedures, and escalation paths. Use when
  writing runbooks for production systems or reviewing operational readiness.
---

# Runbooks

## Purpose

Create runbooks that enable on-call engineers to resolve incidents quickly under stress.

## Key Principle

Written for someone who is **stressed, at 3am, and may not be the author**. Be explicit, sequential, and copy-pasteable.

## Structure

```markdown
# [Alert/Incident Name]

## Severity
[P1/P2/P3] — [Impact description]

## Symptoms
- What the alert says
- What the user experiences

## Quick Diagnosis
1. Check [specific dashboard link]
2. Run: `command to check status`
3. Look for [specific log pattern]

## Common Causes and Fixes

### Cause A: [Description]
**Diagnosis**: [How to confirm this is the cause]
**Fix**:
1. Step-by-step commands
2. Verification that fix worked

### Cause B: [Description]
**Diagnosis**: [How to confirm]
**Fix**: [Steps]

## Escalation
- If not resolved in [time]: escalate to [team/person]
- Contact: [on-call channel/page]

## Post-Incident
- [ ] Update incident timeline
- [ ] Create follow-up ticket for root cause
```

## Writing Principles

- Commands should be copy-pasteable (include full paths, specific values)
- Link to dashboards directly (not "check the dashboard")
- Include expected output of diagnostic commands
- Time-bound each step ("if not resolved in 5 minutes, try next cause")

## Related Skills

- `observability/execution-metrics-alerting` — alerts that trigger runbooks
- `reviewers/review-production-readiness` — runbooks as readiness criterion
