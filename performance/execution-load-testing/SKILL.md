---
name: execution-load-testing
description: >-
  Design and execute load tests covering test scenario design, load profiles,
  success criteria, and result analysis. Use when validating system capacity,
  preparing for traffic events, or establishing performance baselines.
---

# Load Testing

## Purpose

Validate system behavior under expected and peak traffic conditions.

## Workflow

### Step 1: Define Load Profile

| Test type | Purpose | Load pattern |
|-----------|---------|-------------|
| Smoke | Verify test works | Minimal load, short duration |
| Load | Normal conditions | Expected peak traffic, sustained |
| Stress | Find breaking point | Gradually increase until failure |
| Soak | Find memory leaks/degradation | Moderate load, long duration (hours) |
| Spike | Sudden traffic bursts | Sharp increase, then drop |

### Step 2: Design Scenarios

Model real user behavior:
- Realistic request mix (not just one endpoint)
- Think time between requests (users aren't robots)
- Data variety (different payload sizes, query patterns)
- Authenticated vs anonymous traffic ratios

### Step 3: Define Success Criteria

Before running:
- Maximum acceptable p99 latency
- Maximum error rate
- Minimum throughput (requests/second)
- Resource utilization ceiling (CPU <80%, memory <85%)

### Step 4: Execute

- Start with smoke test (verify setup)
- Run against production-like environment
- Isolate from other traffic if possible
- Monitor system metrics during test
- Record results for comparison

### Step 5: Analyze Results

- Plot latency over time (look for degradation)
- Identify saturation point (where latency spikes)
- Check error types at high load
- Compare against success criteria
- Identify the bottleneck (CPU? Memory? DB connections? Network?)

## Checklist

- [ ] Load profile matches expected production traffic
- [ ] Success criteria defined before testing
- [ ] Environment is production-like (data volume, resources)
- [ ] Results baselined for future comparison
- [ ] Bottleneck identified and documented
- [ ] Capacity ceiling known (max safe traffic level)

## Related Skills

- `performance/knowledge-optimization-principles` — interpreting results
- `performance/execution-profiling` — deep-dive on bottlenecks found
- `reviewers/review-production-readiness` — load testing as readiness criterion
