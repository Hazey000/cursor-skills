---
name: knowledge-privacy-by-design
description: >-
  Privacy as an architectural property — data minimization, purpose limitation,
  consent management, GDPR/CCPA engineering implications, data classification,
  retention policies, and right to erasure patterns. Use when designing systems
  that handle PII, implementing data protection features, or reviewing data
  flows for privacy compliance.
---

# Privacy by Design

Privacy is a system quality attribute. Engineer it in from the start — retrofitting is expensive and error-prone.

## Core Principles

### 1. Data Minimization

Collect only what you need, for as long as you need it.

**Questions before adding any data field**:
- What specific feature requires this data?
- Can the feature work with less precise data? (zip code vs. full address)
- Can we derive/aggregate instead of storing raw data?
- What is the retention period? (must be defined before collection)

**Anti-patterns**:
- Collecting "just in case" — no defined purpose
- Logging full request bodies including PII
- Copying production data to development environments

### 2. Purpose Limitation

Data collected for purpose A cannot be used for purpose B without explicit consent.

**Implementation**:
- Tag data with collection purpose in schema metadata
- Enforce purpose checks at the data access layer
- Audit queries against declared purposes
- Separate analytics data from operational data

### 3. Consent Management

```python
class ConsentRecord:
    user_id: str
    purpose: str          # e.g., "marketing_emails", "analytics"
    granted: bool
    granted_at: datetime
    mechanism: str        # e.g., "signup_form_v3", "settings_page"
    withdrawable: bool    # must always be True for GDPR
    withdrawn_at: Optional[datetime]
```

**Rules**:
- Consent must be freely given, specific, informed, unambiguous
- Pre-ticked boxes ≠ consent
- Must be as easy to withdraw as to grant
- Record provenance: when, how, what version of terms
- Children under 16 (or 13 in US): parental consent required

## Data Classification

Classify all data stores and flows:

| Level | Description | Examples | Controls Required |
|-------|-------------|----------|-------------------|
| Public | No restrictions | Marketing content | Integrity only |
| Internal | Business-sensitive | Revenue figures, roadmaps | Access control, audit logs |
| Confidential | PII, customer data | Email, name, address | Encryption at rest + transit, access logs, retention |
| Restricted | Highly sensitive PII | SSN, health data, payment cards | All above + field-level encryption, masking, minimal access |

### PII Identification

Common PII that requires protection:
- **Direct identifiers**: Name, email, phone, SSN, passport, IP address
- **Quasi-identifiers**: DOB + zip + gender (can re-identify via linkage)
- **Sensitive categories** (special treatment): Health, biometrics, race, religion, sexual orientation, political opinion, trade union membership

## Retention Policies

### Design Pattern: TTL-Based Retention

```sql
-- Table design with retention built in
CREATE TABLE user_events (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL,
    event_type TEXT NOT NULL,
    payload JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    retention_until TIMESTAMPTZ NOT NULL  -- Computed at insert based on policy
);

-- Automated cleanup
DELETE FROM user_events WHERE retention_until < NOW();
```

**Retention schedule** (define per data category):

| Data Type | Retention | Rationale |
|-----------|-----------|-----------|
| Access logs | 90 days | Security investigation window |
| Transaction records | 7 years | Financial regulations |
| Marketing preferences | Until consent withdrawn | Purpose-bound |
| Deleted account data | 30 days (soft delete) | Grace period for recovery |
| Session data | 24 hours after expiry | No ongoing purpose |

### Implementation Checklist

- [ ] Every table/collection has a defined retention period
- [ ] Automated jobs enforce retention (don't rely on manual deletion)
- [ ] Backups respect retention (old backups with expired data are purged)
- [ ] Retention is documented and reviewable by data protection officer

## Right to Erasure (Right to Be Forgotten)

### Architecture for Erasure

**Pattern: Indirection via user key**

```
user_data_key (encrypted, per-user)
    ↓ encrypts
user PII in all storage systems

On erasure request:
    → Delete/destroy user_data_key
    → All encrypted PII becomes unrecoverable
    → Non-PII (anonymized analytics) remains intact
```

**Alternative: Distributed erasure pipeline**

```
Erasure Request → Event Bus → 
    ├── User Service (delete profile)
    ├── Order Service (anonymize orders)
    ├── Email Service (delete from lists)
    ├── Analytics (delete raw events, keep aggregates)
    ├── Backups (flag for exclusion on restore)
    └── Third-party APIs (send deletion requests)
```

### Implementation Requirements

1. **Inventory all data stores** containing user data (including logs, caches, backups, third-party systems)
2. **Define erasure scope per store**: delete, anonymize, or retain (with legal basis)
3. **Erasure must complete within 30 days** (GDPR) — design for hours, not days
4. **Provide confirmation** to the user that erasure is complete
5. **Handle edge cases**: pending transactions, legal holds, fraud investigation

### Anonymization vs. Deletion

| Approach | When to Use | Consideration |
|----------|-------------|---------------|
| Hard delete | No retention need | Simplest; irreversible |
| Pseudonymization | Need data for analytics | Replace identifiers with tokens; reversible with key |
| Anonymization | Keep for statistics | Irreversible removal of identifying data; k-anonymity |
| Aggregation | Historical metrics | Roll up individual records into cohort-level data |

**Warning**: Anonymization is harder than it seems. Quasi-identifiers can re-identify. Use k-anonymity (each record is indistinguishable from k-1 others) as minimum bar.

## GDPR Engineering Implications

Not legal advice — engineering concerns only:

| GDPR Right | Engineering Requirement |
|------------|------------------------|
| Right to access (Art. 15) | Export all user data in machine-readable format |
| Right to erasure (Art. 17) | Delete/anonymize across all systems within 30 days |
| Right to rectification (Art. 16) | Allow users to correct their data |
| Right to portability (Art. 20) | Provide data in structured, common format (JSON/CSV) |
| Right to restrict processing (Art. 18) | Flag to halt processing while dispute is resolved |
| Data breach notification (Art. 33) | Detect + notify authority within 72 hours |

### Data Processing Agreements

When using third-party processors (cloud providers, SaaS tools):
- Ensure DPA is in place before sending PII
- Verify processor's data residency matches requirements
- Include right to audit and deletion obligations
- Track which processors hold which data categories

## CCPA Specifics (California)

Key differences from GDPR for engineering:
- **Do Not Sell**: Must implement opt-out mechanism for data sharing with third parties
- **Financial incentives**: Can offer incentives for data collection (not allowed in GDPR)
- **No explicit consent required for collection** (unlike GDPR) — but must disclose
- **Scope**: California residents; businesses meeting revenue/data thresholds

## Privacy in Logging and Monitoring

```python
# WRONG: Logging PII
logger.info(f"User {user.email} logged in from {request.ip}")

# RIGHT: Use opaque identifiers
logger.info(f"User {user.id} logged in", extra={"session": session.id})

# If you must log PII for debugging, use short-lived debug mode with auto-expiry
```

**Rules**:
- Log user IDs, not names/emails
- Mask/truncate sensitive values: `card: ****1234`
- Set short retention on debug-level logs
- Ensure log aggregation systems respect data residency
- Include log data in erasure scope

## Cross-References

- **`knowledge-secure-by-design`** — Encryption, access control, defense in depth for PII
- **`knowledge-owasp-top-10`** — A02 (Cryptographic Failures) for data protection
