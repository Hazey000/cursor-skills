---
name: execution-metrics-alerting
description: >-
  Define SLIs, SLOs, and set up meaningful alerting. Covers metric types,
  cardinality management, alert design, and burn-rate alerting. Use when
  implementing monitoring, designing dashboards, or setting up alerting
  for services.
---

# Metrics and Alerting

## Purpose

Design metrics and alerting that surface real problems without alert fatigue.

## Workflow

### Step 1: Define SLIs (Service Level Indicators)

The four golden signals:

| Signal | Metric | Example SLI |
|--------|--------|-------------|
| Latency | Response time distribution | p99 latency of successful requests |
| Traffic | Request rate | Requests per second |
| Errors | Failure rate | % of requests returning 5xx |
| Saturation | Resource utilization | CPU, memory, queue depth |

### Step 2: Set SLOs (Service Level Objectives)

- Availability: 99.9% = 8.76h downtime/year
- Latency: p99 < 500ms for 99.5% of time windows
- Error rate: < 0.1% of requests

**Start generous, tighten with data.** Premature tight SLOs create noise.

### Step 3: Design Alerts on Burn Rate

Don't alert on instantaneous threshold crossings. Alert on SLO budget consumption rate:

| Urgency | Burn rate | Time window | Response |
|---------|-----------|-------------|----------|
| Page (immediate) | 14x budget burn | 1h | Wake someone up |
| Ticket (soon) | 2x budget burn | 6h | Fix within hours |
| Review (later) | 1x budget burn | 3d | Address this week |

### Step 4: Avoid Cardinality Explosions

High-cardinality labels cause metric storage and query costs to explode:
- Never use user ID, request ID, or email as a metric label
- Bound label values (status codes: fine; arbitrary strings: dangerous)
- Use logs/traces for high-cardinality investigation

### Step 5: Dashboard Design

- One overview dashboard per service (golden signals)
- Drill-down dashboards for investigation
- Time range selector defaulting to useful window (1h or 6h)
- Include deploy markers for correlation

## Checklist

- [ ] Golden signals measured for every service
- [ ] SLOs defined and documented
- [ ] Alerts based on burn rate, not raw thresholds
- [ ] No high-cardinality label dimensions
- [ ] Alert runbook linked from each alert
- [ ] On-call rotation configured for pages

## Related Skills

- `observability/knowledge-observability-pillars` — metrics in context
- `observability/execution-logging-strategy` — correlating metrics with logs
- `reviewers/review-production-readiness` — monitoring as readiness criteria
