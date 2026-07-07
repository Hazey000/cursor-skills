---
name: execution-distributed-tracing
description: >-
  Instrument services for distributed tracing. Covers context propagation,
  span design, sampling strategies, and trace analysis. Use when adding
  tracing to services, debugging cross-service latency, or choosing tracing
  infrastructure.
---

# Distributed Tracing

## Purpose

Step-by-step procedure for instrumenting services to trace requests across boundaries.

## Workflow

### Step 1: Choose Instrumentation Approach

| Approach | Effort | Coverage |
|----------|--------|----------|
| Auto-instrumentation (agent/SDK) | Low | HTTP, DB, common frameworks |
| Manual spans | Medium | Custom business logic |
| Both | Medium | Comprehensive |

Start with auto-instrumentation, add manual spans for business-critical paths.

### Step 2: Context Propagation

Trace context must flow across all boundaries:
- HTTP: `traceparent` / `tracestate` headers (W3C standard)
- Message queues: Trace ID in message metadata/headers
- Background jobs: Pass trace context as job parameter
- Database: Attach trace ID as query comment (for slow query correlation)

### Step 3: Span Design

Good span boundaries:
- Incoming HTTP request (automatic)
- Outgoing HTTP/RPC call (automatic)
- Database query (automatic)
- Business operation (manual): "process order", "validate payment"
- External service call (manual if not auto-detected)

Span attributes to include:
- Service name, operation name
- Error flag and error message if failed
- Relevant business IDs (order_id, user_id)
- Result size or count (for understanding data volume)

### Step 4: Sampling Strategy

| Strategy | Use when | Trade-off |
|----------|----------|-----------|
| Always sample | Low traffic, debugging phase | Storage cost |
| Head-based probabilistic | General production | May miss interesting traces |
| Tail-based (keep errors/slow) | Production at scale | Requires collector buffer |

**Rule**: Always keep error traces and slow traces regardless of sampling rate.

### Step 5: Analysis Patterns

- **Latency waterfall**: Which span is the bottleneck?
- **Error propagation**: Where did the error originate vs where was it reported?
- **Fan-out pattern**: Which parallel calls are slow?
- **Critical path**: Which spans are on the critical (longest) path?

## Checklist

- [ ] Context propagation across all service boundaries
- [ ] Auto-instrumentation for HTTP, DB, common libraries
- [ ] Manual spans for key business operations
- [ ] Sampling strategy appropriate for traffic volume
- [ ] Error traces always captured (not sampled away)
- [ ] Trace ID included in log output for correlation

## Related Skills

- `observability/knowledge-observability-pillars` — tracing in context
- `observability/execution-logging-strategy` — trace ID in logs
- `observability/execution-metrics-alerting` — exemplars linking metrics to traces
