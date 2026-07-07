---
name: execution-threat-modeling
description: >-
  Step-by-step STRIDE-based threat modeling workflow with scoring, prioritization,
  and a reusable template. Use when designing new systems, reviewing architecture
  for security risks, or when asked to perform a threat model on a component
  or system.
---

# Threat Modeling (STRIDE)

Systematic process to identify, score, and mitigate security threats before they become vulnerabilities.

## When to Threat Model

- New system or service being designed
- Significant architectural change to existing system
- New data flow involving sensitive/regulated data
- Integration with external/untrusted systems
- Before security-critical feature development

## Workflow

### Step 1: Decompose the System

Identify and document:

1. **Assets**: What are we protecting? (data, services, credentials, reputation)
2. **Entry points**: Where can external actors interact? (APIs, UI, file uploads, webhooks, message queues)
3. **Trust boundaries**: Where do privilege levels change? (client→server, public→private subnet, service→database)
4. **Data flows**: How does data move between components? (HTTP, gRPC, events, files)
5. **Technologies**: What stack is used? (language, framework, database, cloud services)

**Output**: A data flow diagram (DFD) showing components, data stores, data flows, and trust boundaries.

```
[Browser] --HTTPS--> |TRUST BOUNDARY| --> [API Gateway] --gRPC--> [User Service] --SQL--> [(User DB)]
                                                |
                                          --gRPC--> [Order Service] --SQL--> [(Order DB)]
                                                |
                                          --HTTPS--> [Payment Provider (external)]
```

### Step 2: Identify Threats (STRIDE)

For each component and data flow, systematically check each STRIDE category:

| Category | Threat | Question to Ask |
|----------|--------|-----------------|
| **S**poofing | Identity fraud | Can an attacker pretend to be someone/something else? |
| **T**ampering | Data modification | Can data be modified in transit or at rest without detection? |
| **R**epudiation | Deniability | Can an actor deny performing an action? Do we have audit trails? |
| **I**nformation Disclosure | Data leakage | Can sensitive data be exposed to unauthorized parties? |
| **D**enial of Service | Availability loss | Can the system be made unavailable? |
| **E**levation of Privilege | Unauthorized access | Can a user gain higher privileges than intended? |

**Process**: Walk through each component on the DFD and ask all 6 STRIDE questions. Document every plausible threat.

### Step 3: Score Severity

Use simplified DREAD scoring (1–3 scale for speed):

| Factor | 1 (Low) | 2 (Medium) | 3 (High) |
|--------|---------|------------|----------|
| **D**amage | Minor inconvenience | Data exposure, partial outage | Full data breach, complete outage |
| **R**eproducibility | Complex conditions needed | Reproducible with effort | Trivially reproducible |
| **E**xploitability | Requires insider/physical access | Requires skill + tools | Script kiddie can do it |
| **A**ffected Users | Single user | Subset of users | All users |
| **D**iscoverability | Requires source code access | Discoverable with testing | Publicly visible |

**Score** = sum of all factors (range 5–15)

| Total Score | Priority |
|-------------|----------|
| 12–15 | Critical — fix before launch |
| 8–11 | High — fix in current sprint |
| 5–7 | Medium — schedule in backlog |

### Step 4: Prioritize Mitigations

For each threat above the acceptable risk threshold:

1. **Eliminate**: Remove the feature/component that creates the threat
2. **Mitigate**: Add controls that reduce likelihood or impact
3. **Transfer**: Shift risk to a third party (insurance, managed service)
4. **Accept**: Document as known risk with owner and review date

**Prioritize mitigations by**:
- Score (highest first)
- Cost of mitigation (prefer cheap, structural fixes)
- Coverage (one fix that addresses multiple threats)

### Step 5: Document Residual Risks

After mitigations are planned, document what remains:
- Threats accepted without mitigation (with justification)
- Threats partially mitigated (residual risk level)
- Assumptions that must hold for mitigations to be effective
- Monitoring/detection for accepted risks

## STRIDE Mitigation Patterns

| STRIDE Category | Standard Mitigations |
|-----------------|---------------------|
| Spoofing | Strong authentication, MFA, mTLS, API key rotation |
| Tampering | Input validation, HMAC/signatures, checksums, immutable audit logs |
| Repudiation | Comprehensive audit logging, digital signatures, timestamps |
| Information Disclosure | Encryption (transit + rest), access control, data masking, minimize exposure |
| Denial of Service | Rate limiting, autoscaling, circuit breakers, input size limits, CDN |
| Elevation of Privilege | Least privilege, RBAC/ABAC, input validation, sandboxing, capability dropping |

## Reusable Template

```markdown
# Threat Model: [System/Feature Name]

**Date**: YYYY-MM-DD
**Author**: [Name]
**Reviewers**: [Names]
**Status**: Draft | Reviewed | Approved

## 1. System Overview

[Brief description of the system and its purpose]

### 1.1 Assets
| Asset | Sensitivity | Description |
|-------|-------------|-------------|
| | | |

### 1.2 Entry Points
| ID | Entry Point | Protocol | Auth Required |
|----|-------------|----------|---------------|
| EP1 | | | |

### 1.3 Trust Boundaries
| Boundary | From | To | Controls |
|----------|------|-----|----------|
| TB1 | | | |

### 1.4 Data Flow Diagram

[Diagram or description showing components, flows, and trust boundaries]

## 2. Threats Identified

| ID | Component | STRIDE | Threat Description | Score (DREAD) | Priority |
|----|-----------|--------|-------------------|---------------|----------|
| T1 | | | | /15 | |
| T2 | | | | /15 | |

## 3. Mitigations

| Threat ID | Mitigation | Strategy | Owner | Status |
|-----------|-----------|----------|-------|--------|
| T1 | | Eliminate/Mitigate/Transfer/Accept | | |

## 4. Residual Risks

| Threat ID | Residual Risk | Justification | Review Date |
|-----------|--------------|---------------|-------------|
| | | | |

## 5. Assumptions

- [Assumption 1]
- [Assumption 2]

## 6. Review Schedule

Next review: [Date or trigger condition]
```

## Tips for Effective Threat Modeling

- **Timebox**: 60–90 minutes per session. Multiple short sessions beat one marathon.
- **Right people**: Developer(s) who built it + security-minded reviewer + ops/infrastructure if relevant.
- **Iterate**: Update the threat model when the system changes, not just at initial design.
- **Focus on high-value targets**: Start with the most sensitive data flows and exposed entry points.
- **Use "what if" thinking**: "What if this service is compromised? What if this credential leaks? What if this input is 10GB?"
- **Don't boil the ocean**: A focused threat model on the critical path beats a shallow model of everything.

## Quick-Reference: Per-Component Checks

| Component Type | Key Threats to Check |
|---------------|---------------------|
| Public API endpoint | Injection, AuthN/AuthZ bypass, rate limiting, input validation |
| Database | SQL injection, access control, encryption at rest, backup security |
| File upload | Path traversal, malware, size limits, type validation |
| Authentication flow | Credential stuffing, session fixation, token theft, MFA bypass |
| Third-party integration | SSRF, data leakage, supply chain, availability dependency |
| Message queue | Message tampering, unauthorized publish/subscribe, poison messages |
| Admin interface | Privilege escalation, weak auth, insufficient logging |

## Cross-References

- **`knowledge-owasp-top-10`** — Specific vulnerability categories to check for
- **`knowledge-secure-by-design`** — Architectural principles that mitigate structural threats
- **`knowledge-auth-patterns`** — Authentication/authorization threat mitigations
- **`knowledge-privacy-by-design`** — Data sensitivity classification and PII threat considerations
