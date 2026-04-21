---
# --- Required -------------------------------------------------------
id: platform-ADR-0006
uuid: d4e5f6a7-b8c9-4d0e-1f2a-3b4c5d6e7f8a
memory_type: Decision
title: 'Rejected: Migrate builds service persistence to DynamoDB'
status: rejected
owners:
  - role: builds-lead
    name: builds-team

# --- Recommended ----------------------------------------------------
confidence: reviewed

effective_from: 2025-09-03
effective_to: null

applies_to:
  services: [builds-service]
  domains: [builds]
  systems: []

tags: [dynamodb, postgresql, persistence, builds, rejected]

source_refs:
  - type: meeting
    ref: 'Platform ARB 2025-09-01'
  - type: ticket
    ref: 'BUILDS-0441'

# --- Decision-specific fields ---------------------------------------
decision_question: 'Should the builds service migrate from PostgreSQL to DynamoDB to address its write throughput bottleneck?'
decision_outcome: 'Rejected. DynamoDB migration conflicts with POL-0004 and introduces operational complexity that the identified throughput problem does not justify. The builds team will pursue PostgreSQL optimization and horizontal sharding instead.'

alternatives_considered:
  - option: 'DynamoDB migration (the proposal)'
    reason_rejected: 'Conflicts with POL-0004 (Postgres standard). Operational cost of a second NoSQL database not justified. Throughput problem can be addressed within Postgres.'
  - option: 'PostgreSQL with connection pooling improvements'
    reason_rejected: 'Not yet attempted — designated as the first remediation step before any migration is reconsidered.'
  - option: 'Horizontal Postgres sharding by workspace_id'
    reason_rejected: 'Higher engineering investment but keeps the platform within its operational standard. Designated as fallback if connection pooling alone is insufficient.'

decision_drivers:
  - 'Builds service ingests ~8,000 build event rows/second at peak, causing write contention on the events table'
  - 'P99 event ingest latency spiked to 850ms during a large customer CI run on 2025-08-14'
  - 'POL-0004 requires PostgreSQL for durable persistence; DynamoDB requires an Exception or policy amendment'
  - 'ARB determined the throughput problem is addressable within Postgres before a policy exception is warranted'

approved_by: []

# --- Optional -------------------------------------------------------
implementation_status: not-started
related:
  - uuid: a7b8c9d0-e1f2-4a3b-4c5d-6e7f8a9b0c1d
    relationship: constrained_by
---

## Context

> **This record is rejected.** It is preserved so that the ARB's reasoning is auditable and the DynamoDB proposal is not re-raised without new information.

On 2025-08-14, a large enterprise customer triggered an anomalous CI workload that caused the builds service to write approximately 8,000 build event rows per second for roughly 40 minutes. Write contention on the `build_events` table caused P99 ingest latency to reach 850ms, which cascaded into delayed build status updates and two customer escalations.

The builds team proposed migrating the `build_events` table (and subsequently the broader builds persistence layer) to DynamoDB, citing its write scalability and its known use by large CI systems at other companies.

## Decision

The proposal was rejected by the ARB for two reasons.

**First**, migrating to DynamoDB conflicts with [POL-0004](../rules/platform-POL-0004.md), which requires PostgreSQL for durable persistence. A DynamoDB migration would require either an Exception (which the ARB determined is not warranted before PostgreSQL solutions are exhausted) or a policy amendment (which the ARB is not prepared to consider at this time).

**Second**, the ARB determined that the throughput problem has not yet been addressed with the tools PostgreSQL provides. The `build_events` table has no connection pooling layer, write batching, or partition strategy. These are the standard first responses to PostgreSQL write pressure, and none have been attempted.

The ARB directed the builds team to: (1) instrument and evaluate PgBouncer for connection pooling, (2) batch event writes at the application layer where feasible, and (3) partition the events table by `created_date`. If these measures do not bring P99 latency below 100ms under the observed peak load, the team may re-open a DynamoDB Exception request with the optimization results as evidence.

## Consequences

The builds team must pursue the PostgreSQL optimization path before re-raising the DynamoDB case. This adds estimated 3–5 engineer-weeks of work before an Exception could be reconsidered.

If the optimization path fails, the ARB has signaled it would consider a DynamoDB Exception, but only for the `build_events` write path (not broader persistence), with scope limited to the builds service.

## Related records

This decision was constrained by [POL-0004](../rules/platform-POL-0004.md). Should the builds team return with an Exception request, it must reference both this rejected ADR and the optimization results as part of the Exception justification.
