# Context

## What a Context Is

A Context record captures an environmental fact: _this is true right now (or during a specified period)._ It answers the question "What's the situation?"

Where Decisions capture choices, PolicyRules capture standing guidance, and Exceptions capture sanctioned deviations, Context captures the _world in which all of those happen_. Context is descriptive, not prescriptive. It doesn't tell anyone what to do; it tells them what is.

Examples:

- _The plant-03 OT network is air-gapped from the enterprise network._
- _Commerce is currently running a 12-week migration from MQTT to Kafka._
- _The order-service is the canonical source of truth for order state._
- _Users abandon the signup flow at step 3 because of the SSO handoff._
- _The fulfillment-service has a known 200ms p99 latency spike during handoff windows._

Context is the loosest type in the model, and that's by design. It's the catchall for institutional knowledge that doesn't fit Decisions, Rules, or Exceptions. It's also the type that every role produces (product managers, designers, developers, architects, support) which makes it the type where the shared-backbone model becomes concrete.

---

## How Context Differs from the Other Types

- **Context vs. Decision.** A Decision is a choice; a Context is a fact. "We decided to migrate from MQTT" is a Decision. "MQTT is the current event transport" is Context. The test: could someone have done otherwise? If yes, Decision. If it's just how the world is, Context.
- **Context vs. PolicyRule.** A PolicyRule is prescriptive; a Context is descriptive. "All services must use mTLS" is a PolicyRule. "The production network enforces mTLS at the mesh layer" is Context. If someone violated the record, would it be wrong? If yes, PolicyRule.
- **Context vs. Exception.** An Exception removes a PolicyRule's obligation. A Context describes the environment that might justify an Exception. "The plant floor is air-gapped" is Context. "Plant floor services are exempted from mTLS" is an Exception that cites the Context.

**Rule of thumb:** Context has no obligation attached. Nobody is "required" to comply with it. If a record tells someone what to do, it's not Context.

---

## The Ambiguity Problem

Context is the type people most often get wrong, because the category is broad. Failure modes:

**Everything becomes Context if you squint.** A sloppy writer can frame any fact as Context: "It is a fact that we decided to use Kafka." Well, yes, but that's a Decision. Context _describes the world_, not past choices.

**Context vs. documentation.** A runbook, a how-to, a setup guide: these are operational documentation, not memory bank records. Context records should be facts that an agent might need to _reason over_, not instructions someone follows.

**Transient vs. durable.** Current sprint priority, today's acting manager: these change faster than records can be updated. Use `effective_to` aggressively to bound temporary Context records, and resist capturing things that will be wrong in a week.

**Opinion masquerading as fact.** "The fulfillment team moves too slowly" is not Context. "The fulfillment team has a 4-week release cadence" is. The test: can you point at a verifiable source?

---

## Context-specific Frontmatter

```yaml
---
# --- Base fields (see 01-base-memory-record.md) ---------------------
id: commerce-CTX-0008
uuid: 4d7e2f9b-8c5a-4e3f-9a1d-6b2c8e5f4a7c
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
effective_to: null
source_refs:
  - type: document
    ref: 'Commerce Domain Model v2'

# --- Context-specific fields ----------------------------------------

# Required for Context
context_scope: domain # org | domain | product | program | service | team
fact_statement: >
  The order-service database is the canonical source of truth for order state
  in the commerce domain. All other services consume order state by subscribing
  to the order-events Kafka topic or querying the order-service read API.

# Recommended for Context
# verifiability: how to confirm this is still true
verifiability: >
  Confirmed by the Commerce domain model documentation and by the order-service
  deployment manifest. Can be verified by checking that no other service has
  write access to the orders database.

# assumptions: what the fact rests on
assumptions:
  - 'The order-events topic is available and consumers can subscribe'
  - 'Services that need order state have a path to the read API or Kafka'

# constraints: what this fact implies for work done in its scope
constraints:
  - 'Services should not maintain their own copy of order state'
  - 'Services needing low-latency order state should subscribe to events'
  - 'Any change to the order state schema is a breaking change'

# Optional for Context
fact_type: architectural # architectural | operational | environmental | organizational | market | user-research
confidence_of_fact: high # high | medium | low
last_verified: 2026-03-15
---
```

---

## Field Reference (Context-specific)

### Required

**`context_scope`**: The level at which this fact applies:

- `org`: true across the whole organization
- `domain`: true within a specific domain
- `product`: true for a specific product line
- `program`: true for a specific program or initiative
- `service`: true for a single service
- `team`: true for a specific team

Complements `applies_to`. An org-level Context is retrieved for broad queries; a service-level Context only when that service is in context.

**`fact_statement`**: The fact, in one paragraph or less.

- Good: _"The order-service database is the canonical source of truth for order state."_
- Bad: _"Order state stuff, see body for details."_
- An agent should surface the fact in a sentence or two without reading the body.

### Recommended

**`verifiability`**: How to confirm the fact is still true.

- _What's gained:_ Context rot is the #1 failure mode.[^knowledge-halflife] Without a way to verify, nobody notices when a fact becomes quietly false.
- Point at source documents, configurations, dashboards, test cases. Avoid "ask Jane" — Jane may leave.

**`assumptions`**: Implicit conditions the fact rests on.

- _What's gained:_ When an assumption changes, the Context needs re-verification.
- Think: "if" clauses that make the fact conditional.

**`constraints`**: What this fact implies for work in its scope.

- _What's gained:_ A fact that doesn't constrain anything isn't useful at generation time. Constraints are how Context records influence code generation and review.
- Think of them as the soft version of a PolicyRule: "because X is true, you should probably do Y."

### Optional

**`fact_type`**: Category: `architectural`, `operational`, `environmental`, `organizational`, `market`, `user-research`.

**`confidence_of_fact`**: How sure the fact is accurate: `high`, `medium`, `low`. Distinct from `confidence` (how vetted the record is).

**`last_verified`**: Date the fact was last checked against reality. A fact marked `last_verified: 2022-04-15` is likely stale in 2026.

---

## The Prose Body

```markdown
## Fact

The full statement, expanded from fact_statement if needed.

## Background

Why this fact matters. What would change if it were different?

## Verification

How to check it's still true. Specific dashboards, test cases, docs.

## Assumptions and dependencies

What makes this fact conditional? Prose version of assumptions field.

## Implications

What does this mean for work in scope? Prose version of constraints.

## History

When established? Has it changed? What surfaced it?
```

---

## Context Across Roles

Context is the type that most clearly benefits every role:

- **Architects:** System topology, canonical sources of truth, integration patterns, technical constraints, migration state.
- **Product managers:** Customer constraints, market intelligence, user learnings, roadmap state, commitment facts.
- **Designers:** User research findings, accessibility constraints, usability patterns, persona facts.
- **Developers:** Operational quirks, known issues, performance characteristics, tooling state, vendor behaviors.
- **Support:** Known issues, vendor behaviors, escalation landscape, customer-specific facts.

All roles produce Context records with the same shape but different content. The table below shows how the fields fill for the three personas currently covered by authorship guides:

| Field | Architect | PM / Product Owner | Developer |
| --- | --- | --- | --- |
| `fact_statement` | "The order-service is the canonical source of truth for order state in the commerce domain." | "Enterprise accounts hold 99.95% uptime SLAs in their master service agreements." | "The fulfillment-service has a 200ms p99 latency spike during the nightly reconciliation window." |
| `context_scope` | `domain` or `service` | `segment`, `product`, or `org` | `service` or `environment` |
| `verifiability` | Domain model docs, deployment manifests, platform dashboards | Contract templates (legal), commit dashboards, CS records | Latency dashboards, logs, SLO definitions |
| `constraints` | What downstream services should or shouldn't do given the fact | What product, engineering, or ops must respect given the fact | How callers should behave given the fact (timeouts, windows, SLO exclusions) |
| `tags` | `source-of-truth`, `integration`, `migration-state` | `sla`, `commitment`, `tier`, `market` | `latency`, `performance`, `vendor`, `legacy` |

The authorship trigger: you write a Context when you've learned or confirmed something about your environment that other people (or agents) will need to reason over, and that isn't already captured elsewhere. Not everything worth knowing is worth a record; the test is whether a future teammate or agent would benefit from finding it.

For full worked examples in each voice — plus per-persona authorship triggers, common mistakes, and field-fill cheat sheets — see the per-persona guides:

- [`by-persona/architect-authorship.md`](by-persona/architect-authorship.md)
- [`by-persona/pm-authorship.md`](by-persona/pm-authorship.md)
- [`by-persona/developer-authorship.md`](by-persona/developer-authorship.md)

PM commitments deserve a specific mention: promises made to customers or stakeholders are currently authored as Context records with a `commitment` tag. The open question in the model's roadmap is whether Commitment becomes its own type; until it does, use Context with the tag.

---

## How Agents Should Use Context

**At generation time.** Proactively retrieve Contexts matching the current scope. Use constraints to shape suggestions without hard-blocking.

**At reasoning time.** Before answering, retrieve Context that helps understand the situation. "Can I use MQTT?" + Context "commerce is mid-migration from MQTT" = nuanced answer.

**At query time.** "What's the situation with X?" retrieves matching Context records.

**At deprecation time.** Context can go stale silently. Agents noticing `last_verified` over a year old should warn or flag for review. Without this discipline, the Context repository rots.

---

## Common Mistakes

**Writing opinions as facts.** "The team moves too slowly" is opinion. "The team has a 4-week release cadence" is fact.

**Writing transient things.** Current sprint priority, this week's incident, etc.: these change faster than records update.

**Writing instructions as Context.** Setup guides and runbooks are documentation, not Context.

**Missing `applies_to`.** The most common failure mode. Every Context should answer "what does this apply to?" even if the answer is broad.

**Missing verifiability.** A fact with no verification path rots silently.

**Conflating multiple facts.** "Order-service owns state AND uses Postgres AND has 50ms latency AND is in us-west-2" is four records. Split them.

**Never verifying.** The discipline of periodic verification separates useful Context from stale Context.

---

## Notes

[^knowledge-halflife]: The "half-life of knowledge"—the time until half of what's known in a domain becomes obsolete—comes from Fritz Machlup, _The Production and Distribution of Knowledge in the United States_ (Princeton University Press, 1962). Context records have it worst: environmental facts change without announcement, and unlike rules or decisions, nobody is obligated to update them. The `verifiability` and `last_verified` fields exist to fight this. See [Wikipedia: Half-life of Knowledge](https://en.wikipedia.org/wiki/Half-life_of_knowledge); [AACRAO: Knowledge Decay](https://www.aacrao.org/resources/newsletters-blogs/aacrao-connect/article/knowledge-decay--the-half-life-of-your-education). On organizational forgetting, see [MIT Sloan Management Review: Managing Organizational Forgetting](https://shop.sloanreview.mit.edu/store/managing-organizational-forgetting).
