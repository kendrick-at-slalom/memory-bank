# Memory Bank Scaffold Prompt

> **What this is:** An executable prompt for an AI coding assistant. Copy the content below into a conversation with your agent (Claude Code, Copilot, Cursor, etc.) to interactively scaffold a memory bank for your project.
>
> **Prerequisites:** The agent should have access to this repo's files as reference (clone it or point the agent at it). The agent needs write access to the target project where the memory bank will live.

---

## The Prompt

```markdown
You are going to help me set up a memory bank for my project — a structured knowledge layer that AI coding assistants can query to find decisions, rules, exceptions, and environmental facts about our work.

The memory bank model is documented in this repo. Before we begin, read the following files to understand the model:
- README.md (overview)
- model/README.md (condensed model overview with the authorship map)
- model/00-retrieval-model.md (how agents find records — especially the four-stage funnel)
- model/01-base-memory-record.md (shared schema)

Also note that `model/by-persona/` contains authorship guides for architects, PMs, and developers — same schema, role-specific voice and worked examples. You don't need to read them yet; you'll point me at the right one once you know who will be writing records (Question 2).

Then ask me the following questions ONE AT A TIME, waiting for my answer before proceeding to the next. Use my answers to configure the scaffold.

---

### Question 1: Project context

"Tell me about your project. What does your team build, and roughly how many people contribute? (This helps me choose the right physical organization and scope conventions.)"

---

### Question 2: Roles

"Who will write records in this memory bank? Is it just you, a single team, or multiple roles (e.g., architects, PMs, designers, developers, support)? This determines whether we organize by role, by domain, or as a single flat structure."

Based on the answer:
- Single person or single team → recommend Option C (single repo, folders by type)
- Multiple roles with distinct governance → recommend Option A (per-role repos or directories)
- Multiple roles with shared domain ownership → recommend Option B (per-domain)

Explain your recommendation briefly and ask if they agree or want a different structure.

Then, for any named role that matches one of the persona authorship guides, point them at the corresponding file as onboarding for whoever will write records in that role:
- Architects → `model/by-persona/architect-authorship.md`
- Product managers / product owners → `model/by-persona/pm-authorship.md`
- Developers → `model/by-persona/developer-authorship.md`

If the team has a role that doesn't match an existing persona guide, that's fine — the base schema applies to every role. They'll work from the spec files directly.

---

### Question 3: Exception tracking

"Do you want to track exceptions as first-class records? Exceptions capture sanctioned deviations from your rules — 'we broke the rule intentionally, here's why, here's who approved it.'

I strongly recommend YES. Teams that skip exception tracking accumulate invisible holes in their standards — informal permissions that live in Slack threads, can't be audited, and silently erode governance over time. Exception records force time-bounded, scoped, reviewable deviations instead of permanent shadow rules.

The only case where you might skip this: you're a solo developer with no standing rules to deviate from yet. Even then, you'll likely want it once you have PolicyRules.

Track exceptions? (yes/no)"

Based on answer:
- Yes → scaffold 4 directories (decisions/, rules/, exceptions/, context/)
- No → scaffold 3 directories (decisions/, rules/, context/) and add a note that exceptions fold into Decision records

---

### Question 4: Agent integration

"Which AI coding assistant(s) will consume this memory bank? This determines what instructions file to generate.

Common options:
- Claude Code (uses CLAUDE.md)
- GitHub Copilot (uses .github/copilot-instructions.md)
- Cursor (uses .cursorrules)
- Other / multiple
- Not sure yet (I'll generate a generic instructions file you can adapt)"

---

### Now scaffold the memory bank.

Based on the answers, create the following:

#### 1. Directory structure

Create the appropriate directories based on Question 2 and 3 answers. For a typical single-team setup with exception tracking:

```
memory-bank/
├── decisions/
├── rules/
├── exceptions/
├── context/
└── .gitkeep (in each empty directory)
```

#### 2. Template files

Create one template file per type in a `_templates/` directory:

**`_templates/decision.md`**
```yaml
---
id: <namespace>-ADR-<number>
uuid: <generate-uuid>
memory_type: Decision
title: "<action-oriented title>"
status: proposed
owners:
  - role: <role>
    name: <person-or-team>
applies_to:
  services: []
  domains: []
tags: []
effective_from: <today>
source_refs: []

# Decision-specific
decision_question: "<phrased as a question>"
decision_outcome: "<one sentence>"
alternatives_considered: []
decision_drivers: []
approved_by: []
---

## Context

<what prompted this decision>

## Decision

<what we decided and how it works>

## Alternatives considered

<for each alternative: what it looks like and why rejected>

## Consequences

<positive, negative, neutral>
```

**`_templates/policy-rule.md`**
```yaml
---
id: <namespace>-POL-<number>
uuid: <generate-uuid>
memory_type: PolicyRule
title: "<imperative statement>"
status: proposed
owners:
  - role: <role>
    name: <person-or-team>
applies_to:
  services: []
  domains: []
tags: []
effective_from: <today>
source_refs: []

# PolicyRule-specific
rule_statement: "<clear imperative sentence>"
enforcement: recommended  # required | recommended | advisory
rationale: ""
scope_of_application: ""
exceptions_allowed: true
exception_authority: []
review_cadence: annual
---

## Statement

<full rule text>

## Rationale

<why this rule exists>

## Scope

<what's in, what's out>

## Enforcement

<how this is actually enforced>
```

**`_templates/exception.md`** (if exception tracking is enabled)
```yaml
---
id: <namespace>-EXC-<number>
uuid: <generate-uuid>
memory_type: Exception
title: "<what is exempted from what>"
status: proposed
owners:
  - role: <role>
    name: <person-or-team>
applies_to:
  services: []
  domains: []
tags: []
effective_from: <today>
effective_to: <review-date>
source_refs: []

# Exception-specific
exception_to: <uuid-of-policy-rule>
justification: ""
approved_by: []
review_by: <date>
compensating_controls: []
scope_boundary: ""
---

## Rule being excepted

<brief summary, cite rule id>

## Justification

<what prevents compliance, why constraint exists, what mitigates risk>

## Compensating controls

<detail each control>

## Scope

<in scope, out of scope, explicit boundaries>

## Review and expiration

<when expires, what triggers review>
```

**`_templates/context.md`**
```yaml
---
id: <namespace>-CTX-<number>
uuid: <generate-uuid>
memory_type: Context
title: "<factual statement>"
status: proposed
owners:
  - role: <role>
    name: <person-or-team>
applies_to:
  services: []
  domains: []
tags: []
effective_from: <today>
effective_to: null
source_refs: []

# Context-specific
context_scope: service  # org | domain | product | program | service | team
fact_statement: ""
verifiability: ""
assumptions: []
constraints: []
---

## Fact

<the full statement>

## Background

<why it matters>

## Verification

<how to check it's still true>

## Implications

<what this means for work in scope>
```

#### 3. Agent instructions file

Based on Question 4, generate the appropriate instructions file. Here's the content to adapt per tool:

```markdown
# Memory Bank Instructions

This project maintains a memory bank — structured knowledge records that you should consult when answering questions or generating code.

## When to consult the memory bank

- Before suggesting architectural approaches → check decisions/ and rules/
- Before generating code that touches shared services → check rules/ and exceptions/
- When asked "why is it built this way?" → check decisions/
- When asked "what's the situation with X?" → check context/
- When reviewing code → check rules/ for violations, exceptions/ for sanctioned deviations

## How to query (four-stage funnel)

Always start cheap and narrow progressively:

1. **Glob filenames** — `memory-bank/decisions/*kafka*` or `memory-bank/rules/*persist*`
2. **Grep frontmatter** — `grep "applies_to:.*order-service" memory-bank/` or `grep "^status: accepted" memory-bank/decisions/`
3. **Read frontmatter only** — first 15 lines of surviving candidates (~300 tokens each)
4. **Read full body** — only the 3-5 records that actually matter

Never read all records in full. The funnel exists to keep retrieval cheap.

## When citing records

Always cite the human-readable `id` field (e.g., commerce-ADR-0042) so readers can find the record.

## Record lifecycle

- `status: accepted` = current and authoritative
- `status: proposed` = under discussion, not yet agreed
- `status: superseded` = replaced, follow `superseded_by` link to current version
- `status: deprecated` = no longer applies

## PolicyRule enforcement tiers

- `enforcement: required` = hard constraint, do not violate without an Exception in scope
- `enforcement: recommended` = follow by default, user can override explicitly
- `enforcement: advisory` = mention but don't block
```

#### 4. Namespace convention

Ask: "What namespace prefix should your records use? This appears in IDs like `<namespace>-ADR-0001`. Common choices: project name, team name, or domain name. (e.g., 'commerce', 'platform', 'myapp')"

Use their answer to pre-fill the templates.

#### 5. Seed records (optional)

Ask: "Would you like me to help you write your first few records? Good starting points:

- A Decision you've made recently that someone might ask about later
- A rule your team follows (even informal ones count)
- An environmental fact that shapes your work

Want to seed one or two records now, or start with empty templates?"

If yes, walk them through writing 1-2 records using the templates, filling in real content from their answers. If the user's role matches a persona authorship guide in `model/by-persona/`, read that guide first and use its field-fill cheat sheet and worked examples to shape how you phrase each field — the templates give the structure, the persona guide gives the voice.

---

### After scaffolding, summarize what was created:

- List all directories and files created
- Remind them of the four-stage retrieval funnel
- Point them to the model docs for deeper reference
- Suggest their next step: "Write your first real record. Start with a Decision — they're the most familiar and immediately valuable."
```

---

## Using This Prompt

1. Clone or download this repo
2. Open a conversation with your AI coding assistant
3. Make sure the agent can read the model files (this repo)
4. Paste the prompt above (everything between the triple-backtick code fences)
5. Answer the questions as they come
6. Review what was scaffolded and start writing records

The scaffold creates the structure and templates. The model docs (the numbered files in this repo) are the reference for how to fill them in well.
