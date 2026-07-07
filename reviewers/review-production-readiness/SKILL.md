---
name: review-production-readiness
description: >-
  Comprehensive assessment of whether a system is ready for production traffic.
  Covers observability, reliability, security, operations, and scalability.
  The most thorough reviewer — use before launch or major releases.
---

# Production Readiness Review

Comprehensive evaluation of whether a system is ready to serve production traffic safely, reliably, and observably.

**Cross-references**:
- `observability/knowledge-observability-pillars` for logs, metrics, traces
- `deployment/knowledge-release-strategies` for rollout and rollback patterns

## When to Use

- Before first production deployment of a new service
- Before major releases with significant changes
- After extended outages (validate fixes address root cause)
- Periodic review of production services (quarterly)

## Severity Levels

| Level | Meaning |
|-------|---------|
| **CRITICAL** | Will cause outage, data loss, or security incident — block deployment |
| **WARNING** | Increases incident likelihood or response time — fix before next release |
| **SUGGESTION** | Operational improvement — implement as capacity allows |

## Review Checklist

### 1. Observability

_Can you detect, diagnose, and understand problems in production?_

#### Logging

| Check | Severity if violated |
|-------|---------------------|
| Structured logging (JSON) with consistent field names | WARNING |
| Request correlation IDs propagated across services | WARNING |
| Log levels used correctly (ERROR for failures, WARN for degradation, INFO for business events, DEBUG off in prod) | WARNING |
| No sensitive data in logs (passwords, tokens, full credit card numbers, PII) | CRITICAL |
| Logs aggregated to central system (ELK, CloudWatch, Datadog) | CRITICAL |
| Log retention policy configured | WARNING |

#### Metrics

| Check | Severity if violated |
|-------|---------------------|
| RED metrics exposed: Rate, Errors, Duration per endpoint | CRITICAL |
| System metrics: CPU, memory, disk, network | WARNING |
| Business metrics: key domain events counted | WARNING |
| SLIs defined and measured (latency p50/p95/p99, availability, error rate) | WARNING |
| SLOs documented with error budgets | SUGGESTION |
| Metrics dashboards exist and are accessible to the team | WARNING |

#### Tracing

| Check | Severity if violated |
|-------|---------------------|
| Distributed tracing instrumented across service boundaries | WARNING |
| Trace context propagated in all inter-service calls | WARNING |
| Sampling rate configured appropriately (not 0%, not 100% in high-traffic) | SUGGESTION |

#### Alerting

| Check | Severity if violated |
|-------|---------------------|
| Alerts defined for SLO violations | CRITICAL |
| Alerts on error rate spikes (not just absolute thresholds) | WARNING |
| Alert routing configured to on-call channel (PagerDuty, Opsgenie) | CRITICAL |
| No alert fatigue (alerts are actionable, not noisy) | WARNING |
| Alerts tested with synthetic failures | SUGGESTION |

### 2. Reliability

_Can the system handle failures gracefully?_

#### Health and Lifecycle

| Check | Severity if violated |
|-------|---------------------|
| Health check endpoint exists (`/healthz` or `/health`) | CRITICAL |
| Readiness probe distinct from liveness probe | WARNING |
| Graceful shutdown (drain connections, finish in-flight requests) | CRITICAL |
| Startup probe for slow-starting services | SUGGESTION |
| Health checks verify actual dependencies, not just process alive | WARNING |

#### Failure Handling

| Check | Severity if violated |
|-------|---------------------|
| Circuit breakers on external dependencies | WARNING |
| Retries with exponential backoff and jitter | WARNING |
| Timeouts on all outbound calls (HTTP, DB, queue) | CRITICAL |
| Bulkheads to isolate failure domains (separate thread/connection pools per dependency) | SUGGESTION |
| Fallback behavior defined for degraded dependencies | WARNING |
| Idempotency on write operations exposed to retries | WARNING |

#### Data Safety

| Check | Severity if violated |
|-------|---------------------|
| Database backups automated and tested (restore drill completed) | CRITICAL |
| No single point of failure for critical data | CRITICAL |
| Message queue durability configured (persistent messages, DLQ) | WARNING |
| Data migration rollback tested | WARNING |

### 3. Security

_Is the system hardened for production?_

| Check | Severity if violated |
|-------|---------------------|
| Secrets in vault/env vars, not in code or config files | CRITICAL |
| TLS on all external and inter-service communication | CRITICAL |
| Authentication and authorization enforced on all endpoints | CRITICAL |
| Dependency vulnerability scan passing (no critical/high CVEs) | CRITICAL |
| Container runs as non-root user | WARNING |
| Network policies restrict unnecessary traffic | WARNING |
| Rate limiting on public endpoints | WARNING |
| Security headers configured (CSP, HSTS, X-Frame-Options) | WARNING |
| CORS restrictively configured | WARNING |

### 4. Operations

_Can the team operate this service day-to-day?_

#### Deployment

| Check | Severity if violated |
|-------|---------------------|
| Deployment is automated (CI/CD, not manual steps) | CRITICAL |
| Rollback procedure documented and tested (<5 min to roll back) | CRITICAL |
| Canary or blue-green deployment strategy in place | WARNING |
| Feature flags for risky changes | SUGGESTION |
| Deployment doesn't require downtime | WARNING |
| Smoke tests run post-deployment | WARNING |

#### Documentation

| Check | Severity if violated |
|-------|---------------------|
| Runbook exists covering common failure scenarios | CRITICAL |
| Architecture diagram current | WARNING |
| API documentation complete and up to date | WARNING |
| On-call handoff document with escalation paths | WARNING |
| Known issues / tech debt documented | SUGGESTION |

#### Incident Response

| Check | Severity if violated |
|-------|---------------------|
| On-call rotation established | CRITICAL |
| Team can access production logs, metrics, dashboards | CRITICAL |
| Incident communication channel defined | WARNING |
| Post-incident review process defined | SUGGESTION |

### 5. Scalability

_Can the system handle expected and unexpected load?_

| Check | Severity if violated |
|-------|---------------------|
| Load testing performed at 2x expected peak traffic | WARNING |
| Auto-scaling configured with appropriate min/max | WARNING |
| Resource requests and limits set (CPU, memory) | CRITICAL |
| Database can handle projected data volume for 12+ months | WARNING |
| Connection pools sized for scaled replica count | WARNING |
| No in-process state that prevents horizontal scaling | CRITICAL |
| Rate limiting protects against traffic spikes | WARNING |
| Caching layer can handle cache miss storms | SUGGESTION |
| Queue consumers scale independently of producers | SUGGESTION |

### 6. Compliance and Data

| Check | Severity if violated |
|-------|---------------------|
| PII handling documented and compliant (GDPR, CCPA) | CRITICAL |
| Data retention policies configured | WARNING |
| Audit logging for sensitive operations | WARNING |
| Data classification applied (public, internal, confidential) | SUGGESTION |

## Output Format

```markdown
## Production Readiness Review: [Service/System Name]

**Version/Release**: [version or PR]
**Review Date**: [date]
**Reviewer**: [agent or person]
**Overall**: [READY | READY WITH CAVEATS | NOT READY]

### Findings

#### Observability
##### [CRITICAL] No alerting on error rate
**Issue**: Error rate has no alert — outages go undetected
**Impact**: MTTR increases, SLO breaches go unnoticed
**Fix**: Add alert when error_rate > 1% for 5 minutes, route to PagerDuty

#### Reliability
##### [CRITICAL] No graceful shutdown
**Issue**: Service kills in-flight requests on deploy
**Impact**: Users see intermittent 502 errors during deploys
**Fix**: Handle SIGTERM, drain connections, finish requests within 30s timeout

...

### Summary
| Category | Critical | Warning | Suggestion |
|----------|----------|---------|------------|
| Observability | 1 | 3 | 1 |
| Reliability | 1 | 2 | 1 |
| Security | 0 | 2 | 0 |
| Operations | 1 | 1 | 1 |
| Scalability | 0 | 2 | 1 |
| Compliance | 0 | 1 | 0 |
| **Total** | **3** | **11** | **4** |

### Production Readiness Score

| Category | Score |
|----------|-------|
| Observability | 🟡 Partial |
| Reliability | 🔴 Gaps |
| Security | 🟢 Good |
| Operations | 🟡 Partial |
| Scalability | 🟢 Good |
| Compliance | 🟡 Partial |

### Launch Blockers (must fix)
1. <Critical items that block deployment>
2. ...

### Post-Launch (fix within 30 days)
1. <Warning items>
2. ...

### Improvement Backlog
1. <Suggestions>
2. ...
```

## Quick Pre-Launch Gate

For a fast go/no-go decision, these are the absolute minimum requirements:

| # | Requirement | Status |
|---|-------------|--------|
| 1 | Health check endpoint works | ☐ |
| 2 | Graceful shutdown implemented | ☐ |
| 3 | Logs aggregated and searchable | ☐ |
| 4 | Alerts on error rate and latency | ☐ |
| 5 | Secrets not in code | ☐ |
| 6 | TLS everywhere | ☐ |
| 7 | Auth on all endpoints | ☐ |
| 8 | Rollback tested | ☐ |
| 9 | Runbook exists | ☐ |
| 10 | On-call rotation set | ☐ |

**All 10 must be checked for a GO decision.**
