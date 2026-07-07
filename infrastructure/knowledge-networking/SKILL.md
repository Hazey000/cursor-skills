---
name: knowledge-networking
description: >-
  Networking fundamentals for application developers covering DNS, load
  balancing, CDN, TLS, and service mesh. Decision framework for networking
  patterns. Use when designing distributed systems, troubleshooting connectivity,
  or choosing networking infrastructure.
---

# Networking

## Purpose

Networking knowledge essential for application developers designing distributed systems.

## DNS Essentials

| Record | Purpose | Example use |
|--------|---------|-------------|
| A/AAAA | Domain → IP | Service endpoint |
| CNAME | Domain → domain | Alias to load balancer |
| SRV | Service discovery | Internal service lookup |
| TXT | Metadata | Domain verification, SPF |

**TTL decisions**: Lower TTL = faster failover, more DNS load. Production: 60-300s. During migrations: reduce to 30-60s before cutover.

## Load Balancing

| Layer | Operates on | Use for |
|-------|-------------|---------|
| L4 (TCP/UDP) | IP + port | Raw throughput, protocol-agnostic |
| L7 (HTTP) | URL, headers, cookies | Path routing, header-based routing, TLS termination |

**Algorithm selection**:
- Round robin: Equal-capacity backends
- Least connections: Variable request duration
- Weighted: Heterogeneous capacity
- Consistent hashing: Session affinity, caching

## TLS

- Terminate at edge (load balancer) for simplicity
- mTLS for service-to-service in zero-trust networks
- Certificate rotation: automated (ACME/cert-manager)
- Minimum TLS 1.2, prefer 1.3

## CDN Decision

Use a CDN when:
- Static assets served to geographically distributed users
- API responses that are cacheable and latency-sensitive
- DDoS protection needed at edge

Skip CDN when:
- All users in one region
- All responses are personalized/uncacheable
- Internal-only services

## Related Skills

- `infrastructure/knowledge-cloud-architecture` — networking in cloud context
- `security/knowledge-secure-by-design` — network security principles
- `performance/knowledge-caching-patterns` — CDN as cache layer
