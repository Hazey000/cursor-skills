---
name: knowledge-observability-pillars
description: >-
  The three pillars of observability (logs, metrics, traces) and how they
  interrelate. Decision framework for what to instrument and at what level.
  Use when designing observability for new services or reviewing monitoring
  coverage.
---

# Observability Pillars

## Purpose

Understand how logs, metrics, and traces complement each other and when to use which.

## The Three Pillars

| Pillar | Answers | Granularity | Cost profile |
|--------|---------|-------------|--------------|
| Logs | What happened? (narrative) | Per event | Storage-heavy, high volume |
| Metrics | How is the system performing? (aggregate) | Per time window | Compact, cheap to store |
| Traces | How did a request flow through services? | Per request | Medium volume, high value |

## When to Use Each

| Question | Primary pillar | Supporting |
|----------|---------------|------------|
| "Is the system healthy right now?" | Metrics (dashboards) | — |
| "Why did this specific request fail?" | Traces + Logs | — |
| "Are we meeting our SLOs?" | Metrics (SLI calculations) | — |
| "What changed that caused the regression?" | Metrics (before/after) | Logs (deploy events) |
| "Where is the latency in this request?" | Traces (span waterfall) | Metrics (p99 by service) |

## Correlation Strategy

Connect the three pillars:
- **Trace ID in logs**: Every log line includes the trace ID for the current request
- **Exemplars in metrics**: Link from a metric data point to a representative trace
- **Span events**: Attach structured log events to trace spans

## Instrumentation Priorities

1. **Start with metrics**: SLIs (latency, error rate, throughput) for every service
2. **Add structured logging**: Request lifecycle events with context
3. **Add tracing**: Cross-service request flow (especially when >2 services)
4. **Layer alerting**: Alert on SLO burn rate, not raw metrics

## Anti-Patterns

- **Log-only observability**: Can't aggregate, can't alert proactively
- **Too many metrics**: Cardinality explosion (unbounded label values)
- **Sampling too aggressively**: Missing the interesting (error) traces
- **Alert on every metric**: Alert fatigue, noise drowns signals

## Related Skills

- `observability/execution-logging-strategy` — structured logging implementation
- `observability/execution-metrics-alerting` — SLI/SLO-based alerting
- `observability/execution-distributed-tracing` — tracing implementation
- `reviewers/review-production-readiness` — observability as readiness criteria
