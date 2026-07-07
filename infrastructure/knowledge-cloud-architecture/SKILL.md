---
name: knowledge-cloud-architecture
description: >-
  Cloud architecture principles covering well-architected frameworks, multi-tier
  design, resilience patterns, and cost optimization. Provider-agnostic patterns
  for cloud-native systems. Use when designing cloud deployments, reviewing
  infrastructure architecture, or making cloud service decisions.
---

# Cloud Architecture

## Purpose

Provider-agnostic decision frameworks for designing resilient, cost-effective cloud systems.

## Well-Architected Pillars

| Pillar | Key question | Failure mode |
|--------|-------------|--------------|
| Reliability | Does it recover from failures? | Single points of failure, no redundancy |
| Performance | Does it meet latency/throughput targets? | Under-provisioned, wrong service type |
| Security | Is the attack surface minimized? | Over-permissioned, public exposure |
| Cost | Are you paying for what you use? | Over-provisioned, idle resources |
| Operational excellence | Can you operate it effectively? | No automation, manual processes |

## Architecture Patterns

| Pattern | Use when | Key benefit |
|---------|----------|-------------|
| Multi-AZ | Production workloads | Survives datacenter failure |
| Multi-region | Global users, regulatory, DR | Low latency everywhere, regional failure tolerance |
| Serverless | Variable/spiky traffic, event-driven | Pay per use, no capacity planning |
| Container orchestration | Consistent deployment, team autonomy | Portability, resource efficiency |
| Managed services | Team lacks operational expertise | Reduced ops burden |

## Resilience Patterns

- **Circuit breaker**: Stop calling failing downstream, give it time to recover
- **Bulkhead**: Isolate failures to prevent cascade
- **Retry with backoff**: Handle transient failures
- **Health checks**: Detect and route around unhealthy instances
- **Graceful degradation**: Serve partial functionality when dependencies fail

## Cost Optimization Heuristics

- Right-size before scaling out
- Use spot/preemptible for fault-tolerant workloads
- Reserved capacity for steady-state baseline
- Auto-scale for variable demand
- Delete idle resources (automated detection)
- Data transfer costs often dominate at scale

## Anti-Patterns

- **Lift and shift without redesign**: Running on-prem patterns in cloud (misses cloud-native benefits)
- **Distributed monolith**: Microservices with synchronous dependencies on every request
- **Ignoring data gravity**: Compute far from data = latency + transfer costs
- **Single-AZ production**: One datacenter failure takes down the system

## Related Skills

- `infrastructure/knowledge-twelve-factor` — app design for cloud
- `infrastructure/execution-containerization` — container-based deployment
- `observability/knowledge-observability-pillars` — observing cloud systems
- `reviewers/review-production-readiness` — cloud deployment readiness
