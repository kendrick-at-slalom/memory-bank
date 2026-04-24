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
- guide/README.md (practitioner-facing hydration guide: what a memory bank is, how to organize it, writing a first record)
- model/README.md (condensed schema overview with the authorship map)
- model/00-retrieval-model.md (how agents find records, especially the four-stage funnel)
- model/01-base-memory-record.md (shared schema)

Also note that `guide/by-persona/` contains hydration guides for architects, PMs, and developers; same schema, role-specific voice and worked examples. You don't need to read them yet; you'll point me at the right one once you know who will be writing records (Question 2).

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

- Architects → `guide/by-persona/architects.md`
- Product managers / product owners → `guide/by-persona/pms.md`
- Developers → `guide/by-persona/developers.md`

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
└── .gitkeep (in each empty record directory)

# Step 2 (templates) adds:

memory-bank/\_templates/
├── decision.md
├── policy-rule.md
├── exception.md
└── context.md

# Step 3 (agent integration) writes an agent instructions file

# based on Q4 (e.g., .github/copilot-instructions.md for GitHub Copilot),

# typically at the root of the code repo (not inside memory-bank/).

````

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
````

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

### 3. Agent Instructions File

Based on Question 4, customize and deploy the memory bank starter as the team's instructions file.

**Starter:** `memory-bank/copilot-instructions.starter.md` (in this reference repo). Do not deploy as-is; it contains `<!-- SCAFFOLD:TOKEN -->` markers flagging spots that vary by team.

**Deployment target (based on Q4):**

- GitHub Copilot: `.github/copilot-instructions.md` in the code repo
- Claude Code: `CLAUDE.md` in the code repo root
- Cursor: `.cursorrules` in the code repo root
- Other / not sure: start from `.github/copilot-instructions.md`; user adapts for their tool

**Replacement guide.** For each marker in the starter, apply the replacement rule below based on the Q answers above, then strip the marker comment. After all markers are resolved, strip the top-of-file starter banner too. The deployed file should be clean operational content (the agent reads it every session; every line should earn its keep).

| Marker                               | Replacement rule                                                                                                                                                                                                                                                                                                                                             |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `SCAFFOLD:RECORDS-LAYOUT-START/END`  | Rewrite the wrapped record-directory list to match Q2 (physical organization: per-type, per-role, per-domain) and the memory bank's location/naming. Default assumes colocated `memory-bank/` with per-type subdirs. Adjust paths if the memory bank lives elsewhere (separate repo local, separate repo remote via `gh`, symlinked, custom directory name). |
| `SCAFFOLD:EXCEPTION-START/END`       | Keep the wrapped content if Q3 answered yes to exception tracking. Remove the wrapped content if no. Multiple pairs throughout the file; apply consistently.                                                                                                                                                                                                 |
| `SCAFFOLD:SCOPE-VOCABULARY-HOOK`     | If Q2.5 captured team scope vocabulary (service names, domain names, segments, system IDs), insert a new `## Scope Vocabulary` section at the hook listing those terms. Otherwise, remove the marker and do not insert anything.                                                                                                                             |
| `SCAFFOLD:PERSONA-GUIDES-START/END`  | Keep only the persona links for roles captured in Q2. Adjust path prefixes if the memory bank lives somewhere other than `memory-bank/` in the target repo.                                                                                                                                                                                                  |
| `SCAFFOLD:GUIDE-PATH/TEMPLATES-PATH` | Adjust path prefixes alongside `RECORDS-LAYOUT` based on memory bank location. If templates aren't created (Q3 no OR user opts out), remove the `TEMPLATES-PATH`-wrapped sentence.                                                                                                                                                                           |

After replacements, every `<!-- SCAFFOLD:... -->` comment should be gone. Write the result to the deployment target chosen above.

### 4. Namespace Convention

Ask: "What namespace prefix should your records use? This appears in IDs like `<namespace>-ADR-0001`. Common choices: project name, team name, or domain name. (e.g., 'commerce', 'platform', 'myapp')"

Use their answer to pre-fill the templates.

### 5. Seed Records (optional)

Ask: "Would you like me to help you write your first few records? Good starting points:

- A Decision you've made recently that someone might ask about later
- A rule your team follows (even informal ones count)
- An environmental fact that shapes your work

Want to seed one or two records now, or start with empty templates?"

If yes, walk them through writing 1-2 records using the templates, filling in real content from their answers. If the user's role matches a persona hydration guide in `guide/by-persona/`, read that guide first and use its field-fill cheat sheet and worked examples to shape how you phrase each field. The templates give the structure; the persona guide gives the voice.

---

## After Scaffolding, Summarize What Was Created:

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
```
