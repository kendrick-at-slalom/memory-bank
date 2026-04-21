---
# --- Required -------------------------------------------------------
id: platform-ADR-0001
uuid: a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d
memory_type: Decision
title: Adopt PostgreSQL as the standard relational database
status: accepted
owners:
  - role: platform-architect
    name: platform-architecture-guild

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2025-03-12
effective_to: null

applies_to:
  services: []
  domains: [platform, identity, builds, artifacts]
  systems: []

tags: [postgresql, persistence, database, standards]

source_refs:
  - type: meeting
    ref: 'Platform ARB 2025-03-10'
  - type: document
    ref: 'Platform Engineering Standards v1.0'

# --- Decision-specific fields ---------------------------------------
decision_question: "What relational database should serve as the organization's standard for services requiring structured persistence?"
decision_outcome: 'PostgreSQL 15+ is the standard relational database for all new services requiring relational persistence. Existing services on other databases are grandfathered until their next major version boundary.'

alternatives_considered:
  - option: 'MySQL 8'
    reason_rejected: 'Inferior JSONB and array support; licensing uncertainty introduced by Oracle ownership.'
  - option: 'CockroachDB'
    reason_rejected: 'Operational complexity not justified by current scale; BSL licensing creates export lock-in concerns.'
  - option: 'SQLite'
    reason_rejected: 'Not suitable for production services; single-writer model fails under multi-instance deployments.'

decision_drivers:
  - 'Managed PostgreSQL available across all target cloud regions without additional vendor agreements'
  - 'Team SQL expertise is concentrated in Postgres; migration cost is low'
  - 'Native JSONB support avoids introducing a separate document store for semi-structured data'
  - 'pgvector extension positions platform for ML feature storage without additional infrastructure'
  - 'Mature tooling ecosystem for backup, monitoring, schema migration, and connection pooling'

approved_by:
  - 'Platform Architecture Review Board'

# --- Optional -------------------------------------------------------
implementation_status: complete
review_by: null
---

## Context

In Q4 2024, an audit of service datastores found four different relational databases in production across eight services: PostgreSQL, MySQL, SQLite (in a test harness that had been promoted to production), and a managed CockroachDB cluster spun up by one team without ARB review. Each had its own backup runbook, monitoring configuration, and operational on-call knowledge. When the on-call rotation expanded, gaps appeared: the engineer supporting CockroachDB had left, and the MySQL installation had no backup verification record for seven months.

The ARB chartered a working group to standardize. This decision is the outcome.

## Decision

PostgreSQL 15 or later is the standard relational database for new services. The platform team provides a managed Postgres offering with automated backups, point-in-time recovery, and pre-built Grafana dashboards. New services should use this offering by default rather than self-managing database infrastructure.

Existing services on MySQL, CockroachDB, or SQLite are grandfathered at their current state. They are not required to migrate immediately, but must adopt PostgreSQL at their next major version boundary (a version that requires breaking schema changes or a full redeployment).

## Alternatives considered

**MySQL 8** was the closest competitor. Many engineers have MySQL experience, and the feature set is broadly comparable for simple use cases. The deciding factor was JSONB: stores for configuration data and event payloads increasingly use semi-structured schemas, and Postgres's native JSONB operators are materially more capable than MySQL's JSON support. Oracle's ownership introduced long-term licensing uncertainty the ARB was not willing to accept.

**CockroachDB** offers horizontal scalability that Postgres does not. The ARB evaluated it seriously but concluded the current scale doesn't justify the operational overhead. More importantly, CockroachDB's Business Source License restricts use in competing products — a term that applies to several of the platform's potential use cases.

**SQLite** was evaluated briefly and quickly eliminated. Its single-writer model is incompatible with multi-instance service deployments. Its presence in production was an artifact of a development environment being promoted without review, which the new standard is intended to prevent.

## Consequences

**Positive:** Operational surface area shrinks. One backup strategy, one monitoring stack, one on-call knowledge base for relational persistence. The platform team can provide meaningful managed infrastructure instead of supporting four separate database technologies.

**Negative:** Three existing services need migration plans with defined timelines. Teams that had specific reasons for their current database choices (the CockroachDB team had real scale concerns) will need greenfield conversations about whether those requirements can be met within the standard.

**Neutral:** This decision does not cover non-relational use cases (time-series, full-text search, graph, key-value cache). Those are addressed by separate decisions or outside the scope of this record.

## Related records

This decision was codified into [platform-POL-0004](../rules/platform-POL-0004.md), which establishes the enforcement posture and exception process. The policy record is the authoritative, forward-looking constraint; this ADR is the historical rationale.
