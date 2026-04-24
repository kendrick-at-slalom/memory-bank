---
status: v0 draft
last_updated: 2026-04-23
---

# Hydrating the Memory Bank: Architects

Decisions are the natural entry point. Architects already think in ADRs, and the memory bank's Decision type extends that format without much friction. All four types show up in architect work (PolicyRule for standing patterns, Context for system facts, Exception for sanctioned deviations), but Decisions are where adoption usually starts.

The schema lives in [`model/`](../../model/); this page is the authorship companion for architects.

## When You Author a Record

Architect work produces records at these moments:

| Moment                                                                                                                                          | Record type    |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| You made an architectural decision under constraints (e.g. picked a technology, shaped a boundary, sequenced a migration)                       | **Decision**   |
| You learned or confirmed a system fact (e.g. the `plant-03` OT network is air-gapped, the order-service is the canonical source of truth)       | **Context**    |
| Your team adopted a standing pattern as the way things are done in your scope (e.g. "all persistent services use Postgres")                     | **PolicyRule** |
| You granted or documented a sanctioned deviation from a rule (e.g. a service is exempt from the mTLS policy because of a specific circumstance) | **Exception**  |

The common thread:

- If you're **explaining the _why_ of a system shape** to someone who wasn't in the room, you're most often writing a **Decision** or a **Context**.
- If you're **describing the rules** that govern code being generated inside your scope, you're writing **PolicyRules**.
- If you're recording a **carve-out** from one of those rules, you're writing an **Exception**.

## Field-Fill Cheat Sheet

The schema defines each field; this table shows how you as an architect naturally phrase them.

| Field                     | What you put here                                                                                                                                              |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `decision_question`       | The design question you were answering, framed as a question. "Should the order domain use event sourcing or traditional CRUD persistence?"                    |
| `decision_outcome`        | The choice, one sentence. "Event sourcing using Kafka as the event log."                                                                                       |
| `alternatives_considered` | Technical alternatives with one-sentence rejection reasons. Two or three typical.                                                                              |
| `decision_drivers`        | The constraints that forced the choice. Audit requirements, platform capabilities, prior Decisions, PolicyRules you inherit.                                   |
| `rule_statement`          | An imperative sentence describing the standard. "All services that require durable persistence must use PostgreSQL 15 or later."                               |
| `fact_statement`          | One paragraph stating what is true. "The order-service database is the canonical source of truth for order state."                                             |
| `exception_to`            | The UUID of the PolicyRule being deviated from. Required.                                                                                                      |
| `justification`           | Why the deviation is warranted. Specific technical or operational constraint that prevents compliance, plus what replaces the rule's protection.               |
| `applies_to`              | The scope the record governs. Services, domains, systems. This is what makes your record findable. Thin `applies_to` means the record is invisible to queries. |
| `tags`                    | Topical labels so queries like "show me everything about event sourcing" find your record.                                                                     |

## Worked Examples

### Decision

```yaml
---
id: commerce-ADR-0042
memory_type: Decision
title: Adopt event sourcing for the order domain
status: accepted
owners:
  - role: domain-architect
    name: commerce-architecture-guild
applies_to:
  services: [order-service, fulfillment-service]
  domains: [commerce]
tags: [event-sourcing, kafka, domain-driven-design]
effective_from: 2026-04-10

decision_question: 'Should the order domain use event sourcing or traditional CRUD persistence?'
decision_outcome: 'Event sourcing using Kafka as the event log.'
alternatives_considered:
  - option: 'Traditional CRUD with Postgres'
    reason_rejected: "Doesn't support the audit and replay requirements for order reconciliation."
  - option: 'CQRS without event sourcing'
    reason_rejected: 'Splits the complexity without capturing the audit trail we need.'
decision_drivers:
  - 'Audit trail required by compliance policy POL-0003'
  - 'Replay capability needed for order reconciliation debugging'
  - 'Existing Kafka infrastructure in platform domain'
approved_by:
  - 'Architecture Review Board'
---
```

### Context

```yaml
---
id: commerce-CTX-0008
memory_type: Context
title: Order-service is the canonical source of truth for order state
status: accepted
owners:
  - role: domain-architect
    name: commerce-architecture-guild
applies_to:
  services: [order-service, fulfillment-service, inventory-service]
  domains: [commerce, fulfillment]
tags: [source-of-truth, order-state, canonical]
effective_from: 2024-06-01

context_scope: domain
fact_statement: >
  The order-service database is the canonical source of truth for order state in the commerce domain.
  All other services consume order state by subscribing to the order-events Kafka topic or querying the read API.
verifiability: >
  Confirmed by the Commerce domain model documentation and by the order-service deployment manifest.
  Verifiable by checking that no other service has write access to the orders database.
constraints:
  - 'Services should not maintain their own copy of order state'
  - 'Services needing low-latency order state should subscribe to events'
  - 'Any change to the order state schema is a breaking change'
---
```

### PolicyRule

```yaml
---
id: platform-POL-0017
memory_type: PolicyRule
title: All persistent data services must use Postgres
status: accepted
owners:
  - role: platform-architect
    name: platform-architecture-guild
applies_to:
  domains: [commerce, fulfillment, platform]
tags: [persistence, database, standard]
effective_from: 2025-09-01

rule_statement: 'All services that require durable persistence must use PostgreSQL 15 or later as their primary datastore.'
enforcement: required
rationale: >
  Standardizing on Postgres reduces operational surface area, enables shared tooling for backup,
  monitoring, and schema management, and lets the platform team provide managed database services.
scope_of_application: >
  Applies to all new services in commerce, fulfillment, and platform domains.
  Existing services on other datastores are grandfathered until their next major version.
  Does not apply to caching layers or time-series data.
exceptions_allowed: true
exception_authority:
  - role: platform-architect
review_cadence: annual
---
```

### Exception

```yaml
---
id: plant-floor-EXC-0003
memory_type: Exception
title: Plant-03 telemetry services exempted from mTLS requirement
status: accepted
owners:
  - role: ot-security-lead
    name: plant-floor-security
applies_to:
  services: [telemetry-collector, scada-bridge]
  systems: [plant-03-historian]
  domains: [ot-plant-floor]
tags: [mtls, security, air-gapped]
effective_from: 2026-03-15
effective_to: 2027-03-15

exception_to: 2b4f8a1c-6e3d-4f7a-9b2c-5d8e1a4f7b3c # uuid of the mTLS PolicyRule
justification: >
  Plant-03 historian and telemetry collectors run on an air-gapped network with no external connectivity.
  Certificate distribution requires connectivity to the enterprise PKI, which is unavailable by design.
  Network-layer controls and physical isolation provide equivalent protection.
approved_by:
  - role: ot-security-lead
  - role: platform-arb
review_by: 2027-03-01
compensating_controls:
  - 'Air-gapped network segmentation enforced at physical layer'
  - 'Quarterly physical access review'
  - 'Local authentication with hardware tokens'
---
```

## Common Authorship Mistakes, Architect Edition

- **Writing a Decision as a Context.**
  - "We use Kafka for commerce events" reads like a fact, but if the choice could have gone the other way, it's a Decision.
  - The test: is there a `decision_question` you could frame? If yes, it's a Decision.
- **Writing a PolicyRule when you mean a Decision.**
  - A standing rule applies to many future choices. A single choice, however important, is still a Decision.
  - "We decided to adopt Postgres for the commerce domain" is a Decision; "all persistent services must use Postgres" is the PolicyRule that might come after.
- **Skipping `applies_to` because it feels obvious.**
  - The frontmatter is what an agent uses to find your record. "Obvious from the body" means unfindable.
- **Leaving `alternatives_considered` thin.**
  - If the only alternative is "do nothing," either the decision was foregone or the thinking isn't captured. A future reader (or agent) will assume the alternative was never considered.
- **Exceptions without compensating controls.**
  - An Exception removes a rule's protection. If you can't say what replaces it, you're accepting risk without naming it, which isn't the same as accepting risk knowingly.

## Tips

- Start with your most recent design review. The decision you made there is your first record.
- `applies_to` matters more than any other field for architects. If your record doesn't name the services, domains, or systems it governs, nobody's agent will find it.
- When you write a PolicyRule, think about who might need an Exception against it. If the answer is "probably someone, eventually," make sure the PolicyRule's `id` is something you can reference later.
- Decisions that reference PolicyRules via `constrained_by` are more useful than standalone Decisions. The link tells Copilot which rules shaped the choice.

## Typical Search Patterns

See [Retrieval: Architect Queries](../retrieval.md#architects) for the queries architects most commonly run and how they map to the retrieval funnel.
