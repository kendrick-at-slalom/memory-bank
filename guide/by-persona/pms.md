---
status: v0 draft
last_updated: 2026-04-23
---

# Hydrating the Memory Bank: Product Managers / Product Owners

PMs should start with Context, not Decisions. You produce facts about customers, segments, and commitments that architects and developers depend on at generation time. Those facts are often the most invisible knowledge in the org, living in people's heads or scattered across slide decks. Getting them into structured records is the single best thing a PM can do for the memory bank.

The schema lives in [`model/`](../../model/); this page is the authorship companion for PMs and product owners.

## When You Author a Record

PM work produces records at these moments:

| Moment                                                                                                                                                                                                    | Record type    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| You made a prioritization call under competing asks, or drew a scope boundary, or chose a launch sequence                                                                                                 | **Decision**   |
| You learned or confirmed a customer, market, or segment fact (e.g. enterprise customers hold 99.95% SLAs; tier-2 customers opted out of feature X; the EU market requires a specific localization)        | **Context**    |
| You named a product principle that governs many future choices (e.g. "self-serve features must work without a sales touch," "no paid features behind the free tier's core loop")                          | **PolicyRule** |
| You granted or documented a sanctioned carve-out for a specific customer, region, or release (e.g. this feature ships to customer X without the standard onboarding review because of a prior commitment) | **Exception**  |

The common thread: if you made a call that shapes what gets built, it's a Decision. If you captured a fact about customers or markets that other teams need to know, it's a Context. If you named a product principle that should govern future calls without being re-debated, it's a PolicyRule. If you allowed a deviation from one of those principles for a specific case, it's an Exception.

A special note on **commitments**: if you've promised something to a customer or stakeholder, that lives in the memory bank as a Context today (with a `commitment` tag). The schema's open questions track whether Commitment becomes its own type; for now, fold commitment content into Context and tag it so it's retrievable.

## Field-Fill Cheat Sheet

The schema defines each field; this table shows how you as a PM naturally phrase them.

| Field                     | What you put here                                                                                                                     |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| `decision_question`       | The prioritization or scope call you were making. "Should we ship feature X to tier Y in Q2, or hold it until Q3?"                    |
| `decision_outcome`        | The call, one sentence. "Ship X to tier Y in Q3, not Q2."                                                                             |
| `alternatives_considered` | Scope, timing, or market alternatives with one-sentence rejection reasons.                                                            |
| `decision_drivers`        | The constraints that forced the call. Customer commitments, engineering capacity, market timing, existing Contexts about the segment. |
| `rule_statement`          | The product principle, stated imperatively. "Self-serve features must work end-to-end without a sales touch for tiers 1 and 2."       |
| `fact_statement`          | The customer/market truth, one paragraph. "Enterprise accounts hold 99.95% uptime SLAs written into master service agreements."       |
| `exception_to`            | UUID of the product principle being deviated from. Required.                                                                          |
| `justification`           | Why this customer/region/release is getting the carve-out. Often traces back to a prior commitment or a strategic priority.           |
| `applies_to`              | The scope of the record. Segments, products, markets, regions, customer tiers. What makes it findable.                                |
| `tags`                    | Topical labels. `commitment`, `sla`, `pricing-tier`, `market-entry`.                                                                  |

## Worked Examples

### Decision (a Prioritization call)

```yaml
---
id: product-PRD-0114
memory_type: Decision
title: Delay tier-2 rollout of expense reporting to Q3
status: accepted
owners:
  - role: product-manager
    name: finance-product
applies_to:
  products: [expense-reporting]
  segments: [tier-2]
tags: [scope, rollout, tier-sequencing]
effective_from: 2026-04-22

decision_question: 'Should expense reporting launch to tier-2 customers in Q2 alongside tier-1, or hold until Q3?'
decision_outcome: 'Hold tier-2 rollout until Q3. Launch tier-1 in Q2 as planned.'
alternatives_considered:
  - option: 'Launch both tiers simultaneously in Q2'
    reason_rejected: "Tier-2 requires the SSO integration, which engineering capacity can't deliver in Q2 without slipping tier-1."
  - option: 'Delay both tiers to Q3'
    reason_rejected: 'Breaks the commitment to tier-1 design partners who are expecting Q2 access.'
decision_drivers:
  - 'Commitment to tier-1 design partners captured in CTX-0032'
  - 'Engineering capacity in Q2 fully allocated to tier-1 features'
  - 'Tier-2 customers identified SSO as a blocker in discovery'
approved_by:
  - role: product-lead
    name: finance-product-lead
---
```

### Context (a Customer / Segment fact)

```yaml
---
id: product-CTX-0032
memory_type: Context
title: Enterprise accounts hold 99.95% uptime SLAs in master service agreements
status: accepted
owners:
  - role: product-manager
    name: enterprise-product
applies_to:
  segments: [enterprise]
  products: [core-platform]
tags: [sla, enterprise, commitment, contract]
effective_from: 2025-01-01

context_scope: segment
fact_statement: >
  All enterprise accounts hold a 99.95% uptime SLA written into their master service agreements.
  Availability is measured against the core platform API over rolling 30-day windows.
  Breaches trigger service credits per contract terms and a stakeholder notification from customer success.
verifiability: >
  Confirmed against the enterprise contract template (legal archive) and the availability dashboard
  owned by platform SRE. Any change to the contract template or the measurement approach requires
  this record to be re-verified.
constraints:
  - 'Platform and domain architects: deployment topology must support the SLA'
  - 'Ops: incident response SLA has to match or beat the 30-day window'
  - 'Product: features that could affect availability need SRE sign-off before enterprise launch'
---
```

### PolicyRule (a Product principle)

```yaml
---
id: product-POL-0007
memory_type: PolicyRule
title: Self-serve features must work end-to-end without a sales touch for tiers 1 and 2
status: accepted
owners:
  - role: product-lead
    name: product-leadership
applies_to:
  segments: [tier-1, tier-2]
  products: [core-platform]
tags: [self-serve, onboarding, tiering]
effective_from: 2026-01-15

rule_statement: 'Features exposed to tier-1 and tier-2 customers must complete end-to-end without requiring a sales touch or a human onboarding step.'
enforcement: required
rationale: >
  Self-serve is the economic engine for tiers 1 and 2. Features that silently require a sales-assisted
  onboarding step collapse unit economics and degrade conversion. The rule prevents well-intentioned
  features from eroding the self-serve promise.
scope_of_application: >
  Applies to new features shipped to tier-1 or tier-2. Features exclusive to enterprise (tier-3)
  are out of scope; they can assume a human onboarding step. Migration features for existing
  customers moving between tiers are also out of scope.
exceptions_allowed: true
exception_authority:
  - role: product-lead
review_cadence: semi-annual
---
```

### Exception (a Scope carve-out)

```yaml
---
id: product-EXC-0019
memory_type: Exception
title: Customer-X expense reporting launch without standard onboarding review
status: accepted
owners:
  - role: product-manager
    name: finance-product
applies_to:
  products: [expense-reporting]
  customers: [customer-x]
tags: [commitment, customer-specific, rollout]
effective_from: 2026-05-01
effective_to: 2026-08-01

exception_to: 5a7b2c9e-3f4d-8e1a-6b5c-9d2e7f4a1b8c # uuid of POL-0007
justification: >
  Customer-X holds a written commitment from the prior product lead to receive expense reporting
  by 2026-05-01. The standard onboarding review for tier-2 requires four weeks of review cycles
  that would push the launch past the committed date. The existing customer success manager for
  customer-X is available to support onboarding directly, substituting for the self-serve flow.
approved_by:
  - role: product-lead
  - role: customer-success-director
review_by: 2026-07-15
compensating_controls:
  - 'Customer success manager personally leads onboarding, substituting for the self-serve flow'
  - 'Post-launch check-in at 30 days to flag any gaps that should block future similar exceptions'
  - 'Outcome captured in a follow-up Context record so the precedent is visible'
scope_boundary: >
  Applies only to customer-X's access to expense reporting. Does not extend to other tier-2 customers.
  Does not extend to future launches by customer-X of other products.
---
```

## Common Authorship Mistakes, PM Edition

**Writing a commitment as a Decision.** A Decision is a choice you could have made differently. A commitment is a promise already made. Commitments are Contexts (with a `commitment` tag), not Decisions. You might author a Decision _downstream_ of a commitment ("given CTX-0032, we decided X"), but the commitment itself is the Context.

**Writing a PolicyRule for a one-time scope call.** A prioritization decision for this quarter is a Decision, not a PolicyRule. The PolicyRule is the standing principle that would govern many similar calls. If you find yourself making the same call repeatedly, that's the signal the principle exists and should be written up.

**Writing opinions as Context.** "Enterprise customers care about uptime" is opinion until you ground it. "Enterprise accounts hold 99.95% SLAs in their master service agreements" is a verifiable fact. The `verifiability` field is what separates the two; if you can't fill it in, you're writing an opinion.

**Skipping `applies_to` because the scope feels obvious.** Your PolicyRule that "applies to tiers 1 and 2" needs those tier labels in `applies_to`. An agent filtering for tier-2 records will miss yours otherwise.

**Exceptions without review dates.** A customer-specific carve-out that isn't reviewed becomes permanent. The four-week deviation you granted quietly lives forever. Set `review_by`.

## Tips

For universal practices (frontmatter discipline, lifecycle, cross-role work, verification), see [Leading Practices](../leading-practices.md). The tips below are PM/PO-specific.

- Your Context records are often the _inputs_ to someone else's Decision. A customer commitment you capture today shows up as a `constrained_by` reference in an architect's Decision tomorrow. That's the cross-role link working.
- Commitments with deadlines belong in Context records, not in task trackers. The memory bank captures the _fact_ of the commitment; the tracker manages the _work_ to fulfill it.

## Typical Search Patterns

See [Retrieval: PM / PO Queries](../retrieval.md#pms--pos) for the queries PMs most commonly run and how they map to the retrieval funnel.
