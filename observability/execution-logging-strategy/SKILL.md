---
name: execution-logging-strategy
description: >-
  Structured logging implementation covering log levels, correlation IDs,
  context propagation, and what to log vs what not to log. Use when designing
  logging for services, standardizing log formats, or debugging log-related
  issues.
---

# Logging Strategy

## Purpose

Design a logging approach that enables debugging without overwhelming storage or leaking sensitive data.

## Workflow

### Step 1: Choose Log Format

Use structured logging (JSON) in production:
```json
{"level":"info","ts":"2024-01-15T10:30:00Z","msg":"order created","order_id":"abc123","user_id":"u456","trace_id":"t789"}
```

### Step 2: Define Log Levels

| Level | Use for | Example |
|-------|---------|---------|
| ERROR | Unexpected failures requiring attention | Unhandled exception, data corruption |
| WARN | Recoverable issues, degraded operation | Retry succeeded, fallback used |
| INFO | Significant business events | Order placed, user registered |
| DEBUG | Detailed execution flow (dev/troubleshooting) | Query parameters, cache hit/miss |

**Rule**: Production default is INFO. Enable DEBUG per-service temporarily for investigation.

### Step 3: Contextual Fields

Every log line should include:
- Timestamp (ISO 8601, UTC)
- Level
- Service name
- Trace/correlation ID
- Relevant entity IDs (user, order, etc.)

### Step 4: What NOT to Log

- Passwords, tokens, API keys
- Full credit card numbers
- Personal data (PII) unless necessary and masked
- Request/response bodies in production (too verbose)
- Health check successes (noise)

### Step 5: Correlation

- Generate a correlation/request ID at system entry
- Propagate through all service calls
- Include in every log line for that request
- Pass via header (e.g., `X-Request-ID`) between services

## Checklist

- [ ] Structured format (JSON in production)
- [ ] Consistent fields across services
- [ ] Trace/correlation ID in every log line
- [ ] No sensitive data logged
- [ ] Log levels used appropriately
- [ ] Log rotation / retention configured
- [ ] Logs shipped to aggregation system

## Related Skills

- `observability/knowledge-observability-pillars` — logs in context of other signals
- `observability/execution-distributed-tracing` — trace ID correlation
- `security/knowledge-privacy-by-design` — PII in logs
