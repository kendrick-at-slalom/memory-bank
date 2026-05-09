# ADR 0011: Postgres as the standard relational store

- **Date:** 2026-03-29
- **Status:** Accepted
- **Deciders:** Platform Architecture, Commerce Architecture Guild

## Context

Over the past two years, services in the platform domain have been provisioned against MySQL, MariaDB, Postgres, and (in one case) MS SQL. The proliferation reflects historical team preferences more than any technical justification, and it costs us in three ways:

1. Platform tooling (backup, monitoring, secret rotation) has to handle four engines.
2. On-call engineers have to context-switch between query optimizers and operational quirks.
3. New engineers spend the first quarter learning whichever engine their team picked, instead of the team's domain.

We've informally drifted toward Postgres for net-new services since mid-2025; this ADR codifies that pattern as a standing rule and starts the migration plan for the holdouts.

## Decision

All new services in the platform and commerce domains use Postgres as their relational store unless they have an explicit waiver. Existing services on other engines stay where they are until they hit a major refactor; at that point they move to Postgres.

Waivers are granted only when:

- The service has a demonstrated technical need the engine choice satisfies (e.g., a workload that genuinely needs MySQL's specific replication behavior).
- The waiver is documented as an Exception against this rule, with a review date.

This is a standing rule, not a one-time choice. It applies to services that don't exist yet.

## Alternatives considered

- **Pick a different default (MySQL, MariaDB, etc.).** No technical advantage; the goal is consolidation, not the specific engine. Postgres won on operational maturity and the existing informal preference.
- **Continue ad-hoc.** Fails the consolidation goal.

## Consequences

Positive: consolidated tooling, simpler operations, smaller learning curve for new engineers.

Negative: short-term friction for the three teams currently on non-Postgres engines (no immediate forced migration, but new work pulls toward consolidation).

Neutral: future engine choices for non-relational workloads (key-value, search, time-series) are out of scope for this rule.

## Enforcement

Reviewed at architecture review for any new service. Holdouts on other engines flagged at quarterly platform health review.
