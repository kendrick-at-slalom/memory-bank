# ADR 0007: Event sourcing for the order domain

- **Date:** 2026-03-12
- **Status:** Accepted
- **Deciders:** Commerce Architecture Guild

## Context

The order domain has grown to seven services and a long history of state-shape drift between them. Order reconciliation against fulfillment, billing, and inventory has consistently been our biggest debugging cost: when a discrepancy shows up, we can't replay how we got there because we only persist current state.

Compliance also flagged an audit-trail requirement under the new commerce policy (POL-0003): for every order state transition, we must be able to show who, when, and why for at least 18 months back. Our current append-to-audit-log pattern is brittle and inconsistent across services.

## Decision

The order domain will adopt event sourcing as its persistence strategy. Order state derives from a Kafka event log; reads either replay the log or use snapshots with catchup. New services in the domain will follow the same pattern from day one.

## Alternatives considered

- **Traditional CRUD with a beefier audit table.** Would have addressed the audit requirement but not the reconciliation pain. Rejected.
- **CQRS without event sourcing.** Splits read and write models, but doesn't give us the replay trail. The complexity of maintaining two models without the central event log felt like the worst of both worlds. Rejected.
- **No change; tighten the audit log.** Closest to current state. Doesn't solve replay. Rejected.

## Consequences

Positive: full replay capability, clean audit trail, services can subscribe to events instead of polling each other.

Negative: event schema versioning becomes a discipline we don't currently practice. Onboarding cost for new engineers. Initial migration of three legacy services is non-trivial.

Neutral: snapshot strategy is a future decision we'll need to make once volume forces it.

## References

- POL-0003 (audit trail requirement)
- COMMERCE-1247 (motivating ticket; reconciliation discrepancies in Q4 2025)
