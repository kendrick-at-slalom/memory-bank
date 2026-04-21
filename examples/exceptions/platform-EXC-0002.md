---
# --- Required -------------------------------------------------------
id: platform-EXC-0002
uuid: d0e1f2a3-b4c5-4d6e-7f8a-9b0c1d2e3f4a
memory_type: Exception
title: Identity service session cache exempted from PostgreSQL persistence requirement
status: accepted
owners:
  - role: identity-lead
    name: identity-team

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2025-07-22
effective_to: 2026-07-22

applies_to:
  services: [identity-service]
  domains: [identity]
  systems: []

tags: [redis, postgresql, session-cache, identity, exception]

source_refs:
  - type: meeting
    ref: 'Platform ARB 2025-07-20'
  - type: ticket
    ref: 'IDENT-0238'
  - type: document
    ref: 'Session Cache Load Test Results 2025-07'

# --- Exception-specific fields -------------------------------------
exception_to: a7b8c9d0-e1f2-4a3b-4c5d-6e7f8a9b0c1d

justification: >
  The identity service's session validation SLA is 10ms p99 (CTX-0005). Load testing against
  a PostgreSQL read replica — the compliant solution — produced 18–25ms p99 under realistic peak
  authentication load. Index optimization and query tuning were exhausted; the latency gap is
  structural, not addressable through configuration. Redis 7+ with AOF persistence provides
  4ms p99 for session lookups and meets the SLA. The data in scope is active session state only:
  time-bounded (24h TTL max), non-business-critical, and not subject to audit or compliance
  reporting requirements (those remain in PostgreSQL).
approved_by:
  - role: platform-architect
    name: platform-architecture-guild
  - role: identity-lead
    name: identity-team

# --- Recommended ----------------------------------------------------
review_by: 2026-07-01

compensating_controls:
  - 'Redis AOF (Append-Only File) persistence enabled — session data survives service restart'
  - 'Daily Redis snapshots to S3 with 7-day retention'
  - 'Session TTL maximum of 24 hours — data loss window is bounded'
  - 'All durable session records (audit log, revocation history) remain in PostgreSQL'
  - 'Redis instance monitored with same alerting thresholds as PostgreSQL (availability, memory, replication lag)'

scope_boundary: >
  Applies only to active session state in the identity-service Redis instance. Specifically:
  the session token → user claims mapping used for request authentication. Does not extend to:
  user account records, authentication audit logs, session revocation history, OAuth client
  data, or any other identity data. Does not apply to any other service. Any service wishing
  to use Redis for persistence must file a separate Exception.

# --- Optional -------------------------------------------------------
risk_assessment: mitigated
incident_history: []
related:
  - uuid: e1f2a3b4-c5d6-4e7f-8a9b-0c1d2e3f4a5b
    relationship: depends_on
---

## What this authorizes

This Exception authorizes the identity service to use Redis 7+ with AOF persistence as the primary store for active session state, in place of the PostgreSQL requirement established by [POL-0004](../rules/platform-POL-0004.md).

It does **not** authorize:

- Any other service using Redis for durable persistence.
- The identity service using Redis for anything beyond active session cache.
- This Exception extending beyond the `effective_to` date without a renewal review.

## Justification in detail

Session validation is in the critical path of every authenticated request. The identity service validates session tokens hundreds of thousands of times per second at peak. The 10ms p99 SLA (documented in [CTX-0005](../context/platform-CTX-0005.md)) is a hard requirement — exceeding it degrades authentication latency for every service in the platform.

The identity team conducted load testing against three options:

| Option                                    | P99 latency                   | Compliant with POL-0004 |
| ----------------------------------------- | ----------------------------- | ----------------------- |
| PostgreSQL read replica (index optimized) | 18–25ms                       | Yes                     |
| In-process LRU cache per instance         | 2ms (but invalidation unsafe) | N/A                     |
| Redis 7+ with AOF                         | 4ms                           | No (this Exception)     |

PostgreSQL cannot meet the SLA at the observed peak load. The gap is structural: the overhead of the relational engine and network round-trip for a hot-path query is not reducible to sub-10ms consistently. This conclusion was reviewed by two independent engineers and confirmed by the ARB.

## Why this risk is acceptable

Active session state is structurally different from other durable data:

- **Time-bounded.** Every session has a maximum 24-hour TTL. Data loss exposure is bounded.
- **Not business-critical.** Loss of an active session is a forced logout, not data corruption. Users re-authenticate.
- **Compensated.** AOF persistence and nightly S3 snapshots limit data loss to the AOF sync interval (1 second by default). The Redis instance is monitored identically to PostgreSQL.
- **Durable records in PostgreSQL.** Audit logs and revocation history — the compliance-critical identity data — remain in PostgreSQL and are not affected by this Exception.

## Renewal

This Exception expires **2026-07-22**. Before that date, the identity team must either:

1. Demonstrate that the session latency requirement cannot be met with a compliant solution (no change needed, renewal approved), or
2. Have migrated to a PostgreSQL-based solution that meets the SLA (Exception is closed), or
3. Request a policy amendment to POL-0004 (if this pattern has generalized to multiple services).

A review ticket must be opened in IDENT no later than **2026-07-01** to prevent the Exception from lapsing without decision.
