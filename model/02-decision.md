# Decision

## What a Decision Is

A `Decision` record captures a choice: _we considered options, we picked one, here's why_. It answers the question "Why is it built this way?"

Decisions map onto something most technical teams already know: the Architecture Decision Record (ADR).[^nygard] A Decision record is functionally an ADR with richer frontmatter. Teams who have written ADRs will recognize the shape immediately. Architecture decisions are also a normative concept in [ISO/IEC/IEEE 42010:2022](https://www.iso.org/standard/74393.html)[^iso42010], the international standard for architecture description. The 2022 revision explicitly incorporates architecture decisions as an element of an architecture description, alongside stakeholders, concerns, views, and rationale.

Decisions are the highest-value type for agent-assisted work. When a developer asks "Can I use Redis for this?", the agent needs to find the Decision that said "We chose Postgres over Redis for persistence" and understand both the choice and the reasoning. Without the Decision record, the agent has to infer intent from code, which is exactly what the memory bank exists to prevent.

---

## How Decisions Differ from the Other Types

- **Decision vs. PolicyRule.** A Decision is a **specific choice** at a point in time. A PolicyRule is **standing guidance** for many future choices. "We chose Kafka for the order domain" is a Decision. "All event streaming must use Kafka" is a PolicyRule. Decisions can become PolicyRules when patterns get codified.
- **Decision vs. Context.** A Decision is an **active choice**; a Context is a **passive fact**. "We decided to migrate off MQTT" is a Decision. "The plant floor network only supports TCP" is Context that might shape a decision but isn't itself a choice.
- **Decision vs. Exception.** An Exception is a **sanctioned deviation** from a PolicyRule. "This team is allowed to use Redis despite the Postgres standard" is an Exception, not a Decision.

**Rule of thumb:** if a reasonable person could have made a different choice and this record says which one was picked, it's a Decision.

---

## What a Decision Looks Like for You

Decision maps onto the ADR for architects, but the same schema carries a prioritization call for a PM and an implementation-pattern choice for a developer. Same fields, different content.

| Field | Architect | PM / Product Owner | Developer |
| --- | --- | --- | --- |
| `decision_question` | "Should the order domain use event sourcing or traditional CRUD?" | "Should feature X launch to tier-2 in Q2 or Q3?" | "Which HTTP client library for the fulfillment-service?" |
| `decision_outcome` | "Event sourcing using Kafka." | "Hold tier-2 rollout until Q3." | "reqwest with retry-backoff middleware." |
| `alternatives_considered` | Technical alternatives with rejection reasons | Scope / timing / market alternatives | Other libraries or patterns with practical trade-offs |
| `decision_drivers` | Audit requirements, platform capabilities, prior Decisions, PolicyRules you inherit | Customer commitments, engineering capacity, market timing, Contexts about the segment | Existing infra, team skills, PolicyRules for the stack, performance needs |
| `applies_to` | Services, domains, systems | Segments, products, markets, customer tiers | Services, modules, environments |

The authorship trigger is also different: architects write a Decision when they make an architectural choice under constraints, PMs when they make a prioritization or scope call, developers when they make an implementation choice with reach beyond one file.

For full worked examples in each voice — plus authorship triggers, common mistakes, and field-fill cheat sheets — see the per-persona guides:

- [`by-persona/architect-authorship.md`](by-persona/architect-authorship.md)
- [`by-persona/pm-authorship.md`](by-persona/pm-authorship.md)
- [`by-persona/developer-authorship.md`](by-persona/developer-authorship.md)

---

## Decision-specific Frontmatter

```yaml
---
# --- Base fields (see 01-base-memory-record.md) ---------------------
id: commerce-ADR-0042
uuid: 7f3c9a2e-1b4d-4e8a-9c5f-2d8b1a3e4f5c
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
source_refs:
  - type: meeting
    ref: 'Commerce ARB 2026-04-08'
  - type: ticket
    ref: 'COMMERCE-1247'

# --- Decision-specific fields ---------------------------------------

# Required for Decision
decision_question: 'Should the order domain use event sourcing or traditional CRUD persistence?'
decision_outcome: 'Event sourcing using Kafka as the event log.'

# Recommended for Decision
# alternatives_considered: lets agents explain "why not X?" without re-litigating
alternatives_considered:
  - option: 'Traditional CRUD with Postgres'
    reason_rejected: "Doesn't support the audit and replay requirements for order reconciliation."
  - option: 'CQRS without event sourcing'
    reason_rejected: 'Splits the complexity without capturing the audit trail we need.'

# decision_drivers: the constraints that forced the choice
decision_drivers:
  - 'Audit trail required by compliance policy POL-0003'
  - 'Replay capability needed for order reconciliation debugging'
  - 'Existing Kafka infrastructure in platform domain'

# approved_by: the authority that signed off
approved_by:
  - 'Architecture Review Board'

# Optional for Decision
implementation_status: in-progress # not-started | in-progress | complete | abandoned
---
```

---

## Field Reference (Decision-specific)

### Required

**`decision_question`**: The question the decision answers, phrased as a question.

- Good: _"Should the order domain use event sourcing or traditional CRUD persistence?"_
- Bad: _"Event sourcing in the order domain"_ (title, not question)
- Forces architects to name the real question, which is how agents match future queries to past decisions.

**`decision_outcome`**: The chosen answer, one sentence.

- Good: _"Event sourcing using Kafka as the event log."_
- Bad: _"We talked about it and decided to go with events, details in the body."_
- The atom of organizational memory. An agent should render the outcome without reading the body.

### Recommended

**`alternatives_considered`**: Options that were rejected, each with a reason.

- _What's gained:_ The single most useful Decision field after `decision_outcome`. When someone suggests something that was already considered, the agent says "we looked at that; here's why we didn't" without re-litigating.[^org-learning]
- Format: `{option, reason_rejected}`. Keep reasons short, around one sentence.
- Two or three alternatives is typical. One means the decision was foregone. More than five means you're conflating multiple decisions.

**`decision_drivers`**: Constraints, requirements, or forces that shaped the choice.

- _What's gained:_ How an agent understands _why_ a decision holds. If drivers change, the decision might need revisiting.
- Link to other records when possible: _"Audit trail required by POL-0003"_ is better than _"We need audit trails"_.

**`approved_by`**: Who signed off.

- _What's gained:_ Accountability and trust calibration. Was it a lone engineer, a team, or a formal review body?
- _"Architecture Review Board"_ is fine. _"The team"_ is not useful.

### Optional

**`implementation_status`**: Where the decision is in execution: `not-started`, `in-progress`, `complete`, `abandoned`.

- Distinct from `status`. A decision can be `status: accepted, implementation_status: not-started`: agreed but not started. Or `status: accepted, implementation_status: abandoned`: agreed, started, stopped without formally superseding.

---

## The Prose Body

Suggested section structure (mirrors Nygard ADR format):

```markdown
## Context

What problem or situation prompted this decision? The narrative version of
`decision_question` and `decision_drivers`.

## Decision

What we decided, in prose. The narrative version of `decision_outcome`.
Explain the choice enough that someone could start implementing.

## Alternatives considered

Expanded discussion of alternatives. For each: what it would have looked
like and why it was rejected.

## Consequences

What follows from this decision? Positive, negative, and neutral.
Consequences are often where future Decisions find their seed.

## Related records

Cross-references with explanation of why the relationship matters.
```

This structure is a recommendation. Teams with an existing ADR style can use it as long as the frontmatter is complete. The body is for humans; the frontmatter is what the agent uses.

---

## Common Mistakes

**Conflating multiple decisions into one record.** If `alternatives_considered` has seven entries or `decision_question` has three clauses joined by "and," split them.

**Writing the decision as a statement, not a question.** `decision_question` is a question. Rephrase "We need to use Kafka" as "Which event streaming technology should we use?"

**Thin alternatives.** Listing "do nothing" as the only alternative is a red flag. Either alternatives weren't considered or they weren't written down.

**Missing drivers.** A decision without drivers is a decision without a "why." If you can't articulate the drivers, the decision might be premature.

**Mismatch between `status` and `implementation_status`.** A decision `accepted` for 18 months with `implementation_status: not-started` needs revisiting. `implementation_status: abandoned` with `status: accepted` is organizational debt.

**Forgetting to link superseded records.** Both sides must be updated: the new record sets `supersedes`, the old updates to `status: superseded, superseded_by`.

**Writing Decisions for things that are Context or PolicyRule.** If the record says "normally we do X," it's a PolicyRule. If it describes the environment, it's Context.

---

## Notes

[^nygard]: The ADR format originates from Nygard, M. (2011). ["Documenting Architecture Decisions."](https://www.cognitect.com/blog/2011/11/15/documenting-architecture-decisions) Cognitect Blog. Nygard proposed short text files capturing a single decision with status, context, and consequences. The memory bank extends this with structured frontmatter fields (`decision_question`, `alternatives_considered`, `decision_drivers`) that make decisions machine-queryable — Nygard's original was prose-only. The broader ADR ecosystem has converged around several conventions worth knowing: [adr.github.io](https://adr.github.io/) collects the main variants, [MADR](https://adr.github.io/madr/) provides a Markdown-native template with a modest set of structured fields, and Zimmermann's [Y-statements](https://medium.com/olzzio/y-statements-10eb07b5a177) compress a decision into a single structured sentence ("In the context of X, facing Y, we decided Z and neglected W, to achieve Q, accepting R"). The memory bank's frontmatter is compatible with all three: a Decision record can be rendered as classic Nygard, MADR, or a Y-statement depending on audience.

[^iso42010]: ISO/IEC/IEEE 42010:2022, _Software, systems and enterprise — Architecture description_, is the successor to ISO/IEC 42010:2011 and IEEE 1471. The 2022 revision adds _architecture decision_ as a normative element of an architecture description, a formal recognition of the ADR practice that grew up in industry between 2011 and 2022. See [ISO: 42010:2022](https://www.iso.org/standard/74393.html); [Wikipedia: ISO/IEC/IEEE 42010](https://en.wikipedia.org/wiki/ISO/IEC/IEEE_42010).

[^org-learning]: Teams frequently re-examine previously rejected alternatives when the original evaluation isn't accessible — the `alternatives_considered` field exists to short-circuit that. See Argote, L. (2013). _Organizational Learning: Creating, Retaining and Transferring Knowledge._ Springer. See also [Springer: Organizational Memory](https://link.springer.com/rwe/10.1057/978-1-137-00772-8_210).
