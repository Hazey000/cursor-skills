---
name: execution-profiling
description: >-
  Systematic performance profiling procedure covering CPU profiling, memory
  profiling, I/O analysis, and flame graphs. Use when investigating performance
  issues, establishing baselines, or validating optimization impact.
---

# Profiling

## Purpose

Systematic procedure for finding performance bottlenecks through measurement.

## Workflow

### Step 1: Define the Problem

- What is slow? (specific endpoint, operation, user action)
- How slow? (current measurement)
- What's the target? (acceptable latency/throughput)
- When is it slow? (always, under load, specific conditions)

### Step 2: Reproduce Consistently

- Isolate the slow scenario
- Create a repeatable test case
- Establish baseline measurement

### Step 3: Profile at the Right Level

| Symptom | Profile type | What to look for |
|---------|-------------|-----------------|
| High CPU | CPU profiler / flame graph | Hot functions, tight loops |
| Growing memory | Heap profiler / allocation tracker | Leaks, large allocations |
| Slow responses | Request tracing | Time spent in each phase |
| High latency variance | Latency distribution | P99 outliers |
| Disk/network delays | I/O profiling | Blocking calls, large transfers |

### Step 4: Identify the Bottleneck

Read the profile:
- **Flame graphs**: Wide bars = time spent there. Look for unexpected width.
- **Allocation profiles**: Large/frequent allocations in unexpected places.
- **Trace waterfalls**: Long spans, sequential calls that could be parallel.

### Step 5: Optimize and Verify

1. Change one thing at a time
2. Re-measure with same test case
3. Compare against baseline
4. Verify no regression in other dimensions

## Profiling Principles

- Profile in production-like conditions (data volume, concurrency)
- Profile the actual bottleneck, not what you assume is slow
- Look at distributions (p50, p95, p99), not just averages
- Low-overhead profiling for production; detailed profiling in staging

## Related Skills

- `performance/knowledge-optimization-principles` — decide what to profile
- `performance/execution-load-testing` — profiling under load
- `observability/execution-distributed-tracing` — request-level profiling
