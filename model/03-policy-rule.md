# PolicyRule

## What a PolicyRule Is

A PolicyRule record captures standing guidance[^togaf-principles]: _normally, this is how we do things_. It answers the question "What rules should I follow?"

Where a Decision is a specific choice at a point in time, a PolicyRule is a rule that applies to many future choices. "We chose Kafka for the order domain" is a Decision. "All event streaming must use Kafka unless a documented Exception applies" is a PolicyRule. The Decision is historical; the PolicyRule is forward-looking and imperative.

PolicyRules are the type an agent should consult _proactively_. When a developer asks an agent to generate code that touches event streaming, the agent should find the PolicyRule that says "use Kafka" before suggesting anything else. Decisions explain past choices when asked; PolicyRules constrain future choices without being asked.

---

## How PolicyRules Differ from the Other Types

- **PolicyRule vs. Decision.** A Decision is retrospective ("We picked X"); a PolicyRule is prospective ("From now on, do X"). A Decision can _become_ a PolicyRule when the pattern gets codified. They're closely related but distinct in function.
- **PolicyRule vs. Context.** A PolicyRule is prescriptive ("You must do X"); a Context is descriptive ("X is true"). The test: if someone violated the record, would it be wrong? If yes, PolicyRule. If it's just no longer a fact, Context.
- **PolicyRule vs. Exception.** An Exception is a sanctioned break from a PolicyRule. They come in pairs: you can't have an Exception without the PolicyRule it excepts.

**Rule of thumb:** if the record describes **what someone should do** in the future, it's a PolicyRule.

---

## What a PolicyRule Looks Like for You

The architect's persistence standard, the PM's product principle, and the developer's team practice all fit the same schema. Same fields, different scope.

| Field | Architect | PM / Product Owner | Developer |
| --- | --- | --- | --- |
| `rule_statement` | "All services that require durable persistence must use PostgreSQL 15 or later." | "Self-serve features must work end-to-end without a sales touch for tiers 1 and 2." | "All outbound HTTP calls in fulfillment-service go through the retry-backoff wrapper." |
| `enforcement` | `required` for standards, `recommended` for defaults | `required` for commercial-model principles, `recommended` for patterns | `required` for team practices that block incidents, `recommended` for style |
| `rationale` | Operational surface, shared tooling, governance | Commercial model, unit economics, customer promise | Incident history, centralized behavior, agent guidance at generation time |
| `applies_to` | Domains, services, systems | Segments, products, tiers, markets | Services, modules, repos |
| `exceptions_allowed` | Usually true for standards; false for safety/security hard lines | Usually true (case-by-case carve-outs happen) | True when workarounds are realistic; false for security-shaped rules |

The authorship trigger: you write a PolicyRule when you've named a pattern that should govern many future choices without being re-debated each time. Architects name standing architectural standards; PMs name product principles that bound scope calls; developers name team practices that keep the codebase (and generated code) consistent.

For full worked examples and authorship triggers by persona:

- [`by-persona/architect-authorship.md`](by-persona/architect-authorship.md)
- [`by-persona/pm-authorship.md`](by-persona/pm-authorship.md)
- [`by-persona/developer-authorship.md`](by-persona/developer-authorship.md)

---

## PolicyRule-specific Frontmatter

```yaml
---
# --- Base fields (see 01-base-memory-record.md) ---------------------
id: platform-POL-0017
uuid: 2b4f8a1c-6e3d-4f7a-9b2c-5d8e1a4f7b3c
memory_type: PolicyRule
title: All persistent data services must use Postgres
status: accepted
owners:
  - role: platform-architect
    name: platform-architecture-guild
applies_to:
  domains: [commerce, fulfillment, platform]
  services: [] # empty = applies to all services in listed domains
tags: [persistence, database, standard]
effective_from: 2025-09-01
source_refs:
  - type: meeting
    ref: 'Platform ARB 2025-08-27'
  - type: document
    ref: 'Platform Standards v3.2'

# --- PolicyRule-specific fields -------------------------------------

# Required for PolicyRule
rule_statement: 'All services that require durable persistence must use PostgreSQL 15 or later as their primary datastore.'
enforcement: required # required | recommended | advisory

# Recommended for PolicyRule
# rationale: explains why the rule exists, so agents can reason about applicability
rationale: >
  Standardizing on Postgres reduces operational surface area, enables shared
  tooling for backup, monitoring, and schema management, and lets the platform
  team provide managed database services.

# scope_of_application: more detail than applies_to allows
scope_of_application: >
  Applies to all new services in commerce, fulfillment, and platform domains.
  Existing services on other datastores are grandfathered until their next
  major version. Does not apply to caching layers or time-series data.

# exceptions_allowed: whether Exception records can be filed against this rule
exceptions_allowed: true

# exception_authority: who can approve Exceptions against this rule
exception_authority:
  - role: platform-architect
    name: platform-architecture-guild

# review_cadence: how often the rule should be revisited
review_cadence: annual # none | quarterly | semi-annual | annual | biennial

# Optional for PolicyRule
policy_version: '2.1'
supersedes_policy: null
related_decisions:
  - 7f3c9a2e-1b4d-4e8a-9c5f-2d8b1a3e4f5c
---
```

---

## Field Reference (PolicyRule-specific)

### Required

**`rule_statement`**: The rule itself, in one clear sentence.

- Good: _"All services that require durable persistence must use PostgreSQL 15 or later."_
- Bad: _"We should probably think about standardizing on Postgres."_ (Not a rule.)
- Use imperative language: "must," "should," "must not." Avoid weasel words like "generally" or "typically" unless you mean `enforcement: advisory`.

**`enforcement`**: How strictly the rule is enforced:

- `required`: mandatory. Violations require an Exception record with approval. Agents treat this as a hard constraint when generating code.
- `recommended`: the expected default. Deviations should be documented but don't formally require an Exception. Agents follow the rule unless the user explicitly overrides.
- `advisory`: a suggestion. Teams can deviate freely. Agents mention the rule but don't block on it.

The distinction matters for agent behavior. A `required` rule is a guardrail; the agent refuses to violate it without an Exception in scope. A `recommended` rule is a default. An `advisory` rule is context.[^togaf]

### Recommended

**`rationale`**: Why the rule exists.

- _What's gained:_ Rules without rationale rot. When the underlying reason disappears, the rule keeps being enforced out of habit. Rationale also lets agents reason about applicability to new situations.

**`scope_of_application`**: Natural-language scope description.

- _What's gained:_ Captures nuance that `applies_to` can't: "applies to new services but not existing ones," "applies in production but not dev."

**`exceptions_allowed`**: Boolean. Whether Exception records can be filed.

- _What's gained:_ Some rules admit exceptions (standardization rules); some don't (security rules like "no credentials in source code").

**`exception_authority`**: Who can approve Exceptions against this rule.

- _What's gained:_ Removes ambiguity about who to ask. Same format as `owners`.

**`review_cadence`**: How often the rule should be revisited: `none`, `quarterly`, `semi-annual`, `annual`, `biennial`.

### Optional

**`policy_version`**: Version number for minor revisions that don't warrant supersession.

**`supersedes_policy`**: UUID of an older PolicyRule this one replaces.

**`related_decisions`**: UUIDs of Decisions that established or refined this rule.

---

## The Prose Body

```markdown
## Statement

The full text of the rule if it needs expansion beyond rule_statement.

## Rationale

Why the rule exists. The most important section—it's what lets future
readers evaluate whether the rule still serves its purpose.

## Scope

Who and what it applies to. Be specific about edges.

## Exceptions

How exceptions work. If exceptions_allowed: false, say why.
If true, describe the process.

## Enforcement

How the rule is actually enforced. Tooling? CI/CD gates? Manual review?
A "required" rule with no enforcement mechanism is aspirational.

## History

When established, how it evolved, what prompted creation.
```

---

## How Agents Should Use PolicyRules

**At generation time.** Proactively retrieve PolicyRules matching the current context. `required` rules become hard constraints. `recommended` rules become defaults. `advisory` rules become context.

**At review time.** Check work against applicable PolicyRules. Flag violations unless an Exception exists in scope.

**At query time.** When asked "what are our rules about X?", retrieve by `tags` or `applies_to`, filter by `status: accepted`, group by `enforcement` level.

---

## Common Mistakes

**Writing advisory rules as required.** When everything is `required`, nothing is. Reserve `required` for rules you would actually file an Exception for.

**Weasel-word rule statements.** "Services should generally consider using Postgres where appropriate" is not a rule. Either commit or set `enforcement: advisory`.

**Missing rationale.** Rules without rationale become cargo-culted.

**Ambiguous scope.** "Applies to services in the commerce domain"—Does it apply to existing services? New ones? Both? Grandfathered services? Be specific.

**Unclear exception authority.** `exceptions_allowed: true` with no `exception_authority` means the exception process is undefined.

**Conflating multiple rules.** "Use Postgres, with audit logging, and never store PII in the database" is three rules. Split them.

**Treating rules as permanent.** Rules without `review_cadence` or `review_by` rot silently over time.[^knowledge-decay]

**Not marking rules superseded.** When a new rule replaces an old one, both records must be updated.

---

## Notes

[^togaf]: The enforcement tier system maps onto TOGAF's architecture compliance model, which separates mandatory standards from guidelines and reference architectures. See The Open Group. ["Architecture Governance"](https://pubs.opengroup.org/architecture/togaf9-doc/arch/chap44.html) (TOGAF Standard, Version 9.2, Chapter 44; free Open Group account required). See also ["Architecture Compliance," Chapter 48](https://pubs.opengroup.org/architecture/togaf91-doc/arch/chap48.html). The same graduated-severity pattern shows up in static analysis (error / warning / info) and in RFC 2119's requirement levels (MUST / SHOULD / MAY). See Bradner, S. (1997). ["Key words for use in RFCs to Indicate Requirement Levels."](https://datatracker.ietf.org/doc/html/rfc2119) RFC 2119.

[^togaf-principles]: PolicyRules follow the same shape as TOGAF Architecture Principles, which use a four-part template: **Name, Statement, Rationale, Implications**. The memory bank's `title`, `rule_statement`, `rationale`, and `scope_of_application` + body `Consequences` sections map onto that template directly. Teams already maintaining a TOGAF Principles Catalog can port principles into the memory bank without restructuring them. See The Open Group. ["Architecture Principles"](https://pubs.opengroup.org/architecture/togaf9-doc/arch/chap23.html) (TOGAF Standard, Version 9.2, Chapter 23; free Open Group account required).

[^knowledge-decay]: The concept of the "half-life of knowledge" (the time until half of what's known in a domain becomes obsolete) was introduced by Fritz Machlup in _The Production and Distribution of Knowledge in the United States_ (Princeton University Press, 1962). It applies to organizational rules as much as to facts: a rule whose rationale has disappeared is being enforced out of inertia. See [Wikipedia: Half-life of Knowledge](https://en.wikipedia.org/wiki/Half-life_of_knowledge).
