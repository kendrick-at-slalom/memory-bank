---
# --- Required -------------------------------------------------------
id: platform-ADR-0007
uuid: e5f6a7b8-c9d0-4e1f-2a3b-4c5d6e7f8a9b
memory_type: Decision
title: Use Redis for identity service session cache
status: accepted
owners:
  - role: identity-lead
    name: identity-team

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2025-07-22
effective_to: null

applies_to:
  services: [identity-service]
  domains: [identity]
  systems: []

tags: [redis, caching, session, identity, postgresql]

source_refs:
  - type: meeting
    ref: 'Platform ARB 2025-07-20'
  - type: ticket
    ref: 'IDENT-0238'

# --- Decision-specific fields ---------------------------------------
decision_question: 'What mechanism should the identity service use to store active session state, given its 10ms p99 read latency SLA?'
decision_outcome: 'Redis 7+ with AOF persistence for active session cache. An Exception to POL-0004 (EXC-0002) authorizes this deviation from the PostgreSQL standard. Durable session records (audit log, revocation history) remain in PostgreSQL.'

alternatives_considered:
  - option: 'PostgreSQL with read replica'
    reason_rejected: 'Load testing showed read replica P99 latency of 18–25ms for session lookups under peak authentication load. Cannot meet the 10ms SLA documented in CTX-0005.'
  - option: 'In-process LRU cache per service instance'
    reason_rejected: 'Cache invalidation across instances on logout or revocation is not solvable without a shared coordination layer — which is effectively Redis.'
  - option: 'Memcached'
    reason_rejected: 'No persistence option; session loss on restart is unacceptable. Redis AOF provides the durability floor we need.'

decision_drivers:
  - 'Identity service session validation SLA is 10ms p99 (CTX-0005)'
  - 'PostgreSQL read replica benchmarked at 18–25ms p99 for session lookups under peak load'
  - 'Session data is not durable business data — 24h TTL and AOF persistence are sufficient durability'
  - 'Redis is already operated by the platform team for other caching workloads'
  - 'Exception EXC-0002 provides sanctioned authorization for this deviation from POL-0004'

approved_by:
  - 'Platform Architecture Review Board'
  - 'Identity Team Lead'

# --- Optional -------------------------------------------------------
implementation_status: complete
related:
  - uuid: a7b8c9d0-e1f2-4a3b-4c5d-6e7f8a9b0c1d
    relationship: constrained_by
  - uuid: d0e1f2a3-b4c5-4d6e-7f8a-9b0c1d2e3f4a
    relationship: relates_to
---

## Context

Authentication is in the critical path of every request to the platform. When a user's session token is presented, the identity service must validate it — check expiry, check revocation status, retrieve the associated user claims — before the request can proceed. This validation happens hundreds of thousands of times per second at peak.

[CTX-0005](../context/platform-CTX-0005.md) documents the SLA: session validation must complete in 10ms p99. Load testing against a PostgreSQL read replica showed consistent 18–25ms p99 for the session lookup query under realistic authentication load — not close enough to the target to justify optimizing further within Postgres.

## Decision

Redis 7+ with Append-Only File (AOF) persistence is the session cache for the identity service. Sessions are written to Redis on creation with a TTL matching the session expiry (maximum 24 hours). Validation reads from Redis; revocation writes to both Redis (immediate TTL invalidation) and PostgreSQL (durable audit record).

Durable records — the audit log of session creation and revocation events, the long-term session history used for compliance reporting — remain in PostgreSQL. Redis holds only the active, time-bounded session state.

This decision operates under the authorization of [EXC-0002](../exceptions/platform-EXC-0002.md), which provides a sanctioned Exception to [POL-0004](../rules/platform-POL-0004.md)'s PostgreSQL requirement. EXC-0002 is time-bounded and must be reviewed annually.

## Alternatives considered

**PostgreSQL with read replica** was the first option tested. It failed on latency. Even with proper indexing and the session lookup query optimized to use a single covering index, the P99 consistently exceeded 18ms under peak load. The SLA is 10ms. This is not a tuning problem — it reflects the overhead of the relational engine and network round-trip for a hot-path query.

**In-process LRU cache** was considered as a lower-infrastructure alternative. The problem is invalidation. When a user logs out or when an administrator revokes a session, every service instance currently holding that session in its local cache is stale until the TTL expires or the process restarts. For a security-sensitive operation like session revocation, this window is unacceptable.

**Memcached** provides similar read latency to Redis but has no persistence story. A service restart would invalidate all active sessions, requiring every user to re-authenticate. Given the scale of the identity service, this would be a noticeable user-facing event every time the service is updated.

## Consequences

**Positive:** Session validation latency dropped to 4ms p99 in load testing, meeting the SLA with headroom. Logout and revocation are immediate and consistent across all service instances.

**Negative:** The platform now operates two persistence technologies for the identity domain. The Redis instance must be monitored, backed up (snapshots to S3), and available in the on-call runbook. The Exception (EXC-0002) must be reviewed annually to confirm conditions haven't changed.

**Neutral:** The decision is scoped to active session cache only. Any future need for Redis in other services would require separate ARB review.

## Related records

- [POL-0004](../rules/platform-POL-0004.md): The PostgreSQL standard this decision deviates from.
- [EXC-0002](../exceptions/platform-EXC-0002.md): The Exception authorizing the deviation.
- [CTX-0005](../context/platform-CTX-0005.md): The 10ms SLA that drove the decision.
