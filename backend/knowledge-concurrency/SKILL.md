---
name: knowledge-concurrency
description: >-
  Concurrency principles covering threads, async, locks, race conditions, and
  coordination patterns. Decision framework for choosing concurrency models.
  Use when designing concurrent systems, debugging race conditions, or choosing
  between sync and async approaches.
---

# Concurrency

## Purpose

Decision frameworks for concurrency model selection and safe concurrent code.

## Concurrency Models

| Model | Best for | Trade-off |
|-------|----------|-----------|
| Thread-per-request | CPU-bound, simple mental model | Memory overhead, context switches |
| Event loop (async) | I/O-bound, many connections | Can't do CPU work, callback complexity |
| Actor model | Independent stateful entities | Message overhead, debugging complexity |
| CSP (channels) | Pipeline-style processing | Channel capacity management |
| Fork-join | Parallelizable computation | Overhead for small tasks |

## Decision Framework

1. **Is the work I/O-bound or CPU-bound?**
   - I/O → async/event loop
   - CPU → thread pool / parallel
2. **Do workers need shared state?**
   - No → embarrassingly parallel, simplest model
   - Yes → choose coordination carefully
3. **What's the concurrency level?**
   - Thousands of connections → async (not thread-per-request)
   - Dozens of workers → threads are fine

## Race Condition Prevention

**Hierarchy of preference** (best to worst):
1. **Eliminate shared state** — immutable data, message passing
2. **Confine state** — each thread owns its data exclusively
3. **Synchronize access** — locks, atomics (last resort)

## Common Concurrency Bugs

| Bug | Cause | Prevention |
|-----|-------|------------|
| Race condition | Unsynchronized shared state | Immutability, locks, atomics |
| Deadlock | Circular lock dependency | Lock ordering, timeouts |
| Starvation | Unfair scheduling | Fairness policies, bounded queues |
| Thundering herd | All waiters wake simultaneously | Jitter, leader election |

## Safe Patterns

- **Immutable shared data**: Safe to read from any thread
- **Thread-local storage**: No sharing, no races
- **Producer-consumer queues**: Bounded, backpressure-aware
- **Compare-and-swap (CAS)**: Lock-free updates for simple values

## Anti-Patterns

- Sharing mutable state without synchronization
- Holding locks during I/O operations
- Unbounded queues (eventual OOM)
- `sleep()` as synchronization

## Related Skills

- `backend/execution-background-jobs` — concurrent job processing
- `performance/knowledge-optimization-principles` — concurrency for performance
