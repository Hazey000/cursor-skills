---
name: execution-background-jobs
description: >-
  Design reliable background job processing systems. Covers job queue selection,
  idempotency, retry strategies, dead letter queues, and monitoring. Use when
  implementing async processing, task queues, or scheduled jobs.
---

# Background Jobs

## Purpose

Step-by-step procedure for designing reliable background job processing.

## Workflow

### Step 1: Classify the Job

| Type | Characteristics | Example |
|------|----------------|---------|
| Fire-and-forget | Best effort, failure acceptable | Analytics event |
| At-least-once | Must complete, duplicates acceptable | Send email |
| Exactly-once | Must complete, no duplicates | Payment processing |

### Step 2: Design for Idempotency

Every job MUST be safe to retry. Techniques:
- Use unique idempotency keys
- Check if work already done before starting
- Use database transactions with unique constraints
- Design operations as convergent (reaching same state regardless of retries)

### Step 3: Define Retry Strategy

- Exponential backoff with jitter
- Maximum retry count (typically 3-5 for transient errors)
- Dead letter queue for exhausted retries
- Different retry policies for different error types (don't retry validation errors)

### Step 4: Handle Failure

- Dead letter queue for investigation
- Alerting on DLQ growth
- Manual retry/replay capability
- Poison message isolation (don't let one bad message block the queue)

### Step 5: Monitor

- Queue depth (growing = processing too slow)
- Processing latency (p50, p99)
- Failure rate
- DLQ size
- Job duration

## Checklist

- [ ] Jobs are idempotent
- [ ] Retry strategy with exponential backoff
- [ ] Dead letter queue configured
- [ ] Timeouts on job execution
- [ ] Monitoring on queue depth and failure rate
- [ ] Graceful shutdown (finish current job before stopping)
- [ ] Job payload is self-contained (no external state dependency)

## Related Skills

- `backend/knowledge-concurrency` — concurrency models for job processing
- `observability/execution-metrics-alerting` — monitoring job systems
- `architecture/knowledge-event-driven` — event-driven job triggers
