---
id: commerce-ADR-0042
uuid: 7f3c9a2e-1b4d-4e8a-9c5f-2d8b1a3e4f5c
memory_type: Decision
title: Adopt event sourcing for the order domain
status: proposed
owners:
  - role: domain-architect
    name: commerce-architecture-guild
applies_to:
  services: [order-service, fulfillment-service]
  domains: [commerce]
tags: [event-sourcing, kafka, audit, persistence]
effective_from: 2026-03-12
source_refs:
  - type: adr
    ref: docs/adrs/0007-event-sourcing-orders.md
  - type: ticket
    ref: COMMERCE-1247

decision_question: "Which persistence strategy should the order domain use to satisfy audit and replay requirements?"
decision_outcome: "Event sourcing with Kafka as the event log. New services in the domain follow the pattern from day one."
alternatives_considered:
  - option: "Traditional CRUD with a beefier audit table"
    reason_rejected: "Addresses the audit requirement but not the reconciliation pain."
  - option: "CQRS without event sourcing"
    reason_rejected: "Splits read and write models without giving us a replay trail."
  - option: "No change; tighten the audit log"
    reason_rejected: "Doesn't solve the replay problem."
decision_drivers:
  - "Audit trail required by POL-0003"
  - "Replay capability needed for order reconciliation"
  - "Existing Kafka infrastructure available in platform domain"
approved_by:
  - "Commerce Architecture Guild"
implementation_status: in-progress
---

## Context

The order domain has grown across seven services with state-shape drift between them. Reconciliation against fulfillment, billing, and inventory is the largest debugging cost we have because we only persist current state, not how we got there. Compliance flagged an audit-trail requirement under POL-0003 that the existing append-to-audit-log pattern doesn't satisfy.

## Decision

The order domain adopts event sourcing as its persistence strategy. Order state derives from a Kafka event log; reads replay the log or use snapshots with catchup. New services in the domain follow this pattern from day one.

## Alternatives considered

See `alternatives_considered` field.

## Consequences

Positive: replay capability, clean audit trail, services subscribe to events instead of polling.

Negative: event schema versioning becomes a discipline. Onboarding cost. Migration of three legacy services is non-trivial.

Neutral: snapshot strategy is a future decision once volume forces it.

## Related records

- POL-0003 (audit trail requirement) via `constrained_by`
