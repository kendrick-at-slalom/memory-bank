# Memory Bank Scaffold Prompt

> **What this is:** An executable prompt for an AI coding assistant. Copy the content below into a conversation with your agent (Claude Code, Copilot, Cursor, etc.) to interactively scaffold a memory bank.
>
> **Prerequisites:** The agent needs access to this repo's files as reference (clone it or point the agent at it) and write access to the target project (or target memory bank repo) where the bank will live.

---

## How to Use This Prompt

1. Clone or download this repo.
2. Open a conversation with your AI coding assistant.
3. Make sure the agent can read this repo's files (the scaffold references them).
4. Copy everything inside the four-backtick fence below (`The Prompt` section) into the agent chat.
5. Answer the questions as they come. The prompt asks the agent to use clickable choices wherever possible; if your tooling supports `AskUserQuestion`-style selection you'll click through, otherwise you'll reply with numbers.
6. Review what was scaffolded and start writing records.

The scaffold creates the structure, templates, and agent instructions file. The model docs in `model/` and the hydration guide in `guide/` are the reference for how to fill records well.

The interview leads with a scope question (project / team / domain / org / larger). Scope is the most consequential choice in the setup because it determines how much cross-cutting knowledge ends up in one bank versus fragmented across many.

---

## The Prompt

````markdown
You are going to help me set up a memory bank: a structured knowledge layer that AI coding assistants can query to find decisions, rules, exceptions, and environmental facts about the scope it serves.

Before we begin, read the following files in this reference repo:

- `README.md` (overview)
- `guide/README.md` (practitioner-facing hydration guide: what a memory bank is, how to organize it, writing a first record)
- `model/README.md` (condensed schema overview with the authorship map)
- `model/00-retrieval-model.md` (how agents find records, especially the four-stage funnel)
- `model/01-base-memory-record.md` (shared schema)

Also note that `guide/by-persona/` contains hydration guides for architects, PMs, and developers (same schema, role-specific voice and worked examples). You don't need to read them yet; you'll point me at the right one once you know who will be writing records.

## How to run this interview

Ask me each question ONE AT A TIME, waiting for my answer before proceeding to the next. **Whenever a question has enumerable options, present them using `AskUserQuestion` (or your tooling's equivalent clickable-choice interface). If no clickable UI is available, present options as a numbered list and ask me to reply with the number.** For questions that are genuinely freeform, ask freeform.

Use my answers to configure the scaffold. If an answer sounds ambiguous, ask a short clarifying follow-up.

The first question is about **bank scope**; it's the most consequential choice in the setup. Be intentional there, and gently push back if I pick the narrowest option out of habit rather than because the scope genuinely is that narrow.

---

### Question 1: Bank scope

The memory bank model works at any scope: project, team, domain, org, cross-org. Broader is often better when governance supports it: broader banks reduce fragmentation and let cross-cutting knowledge (PolicyRules, Context, Decisions that span teams) be authored once and found by everyone. Narrow enough to match the actual governance boundary is the target; don't pick narrower than necessary.

(Clickable.) "What scope will this memory bank serve?"

Options:

1. **A single project or repo.** Best when the project is genuinely self-contained and won't share knowledge with sibling projects.
2. **A team.** One governance, may span multiple projects or repos the team owns.
3. **A domain.** Cross-team scope (e.g., "commerce", "platform", "OT plant floor"). Multiple teams contribute under domain-level governance.
4. **An org or division.** Cross-domain scope. Strong governance needed; benefits are large for cross-domain learning.
5. **Something else (I'll describe).**

If I pick option 1, probe briefly: "Is there a team-scope or domain-scope bank that could serve this project alongside siblings? Per-project banks fragment knowledge that usually wants to be shared." Accept the answer either way; don't push hard.

---

### Question 2: Scope context

(Freeform.) "Tell me about the scope you picked in a few sentences. What does it cover and why did you draw the boundary there? (This helps me choose sensible defaults and name things well.)"

Then ask (clickable) how many people contribute at this scope:

Options:

1. 1–5 (solo or small team).
2. 5–50 (mid-sized team, small domain, or small org).
3. 50+ (domain, large team, division, or org).

---

### Question 3: Memory bank location

(Clickable.) "Where will the memory bank live?"

Options:

1. **Colocated**: inside the same repo as the code (works when the bank's scope matches one repo, e.g., a project-scoped bank inside a project repo).
2. **Separate repo, cloned locally**: memory bank is its own repo, cloned alongside the code repo(s) it serves.
3. **Separate repo, accessed remotely via `gh`**: memory bank lives in a GitHub repo; agents fetch content via `gh` CLI without a local clone.
4. **Symlinked**: memory bank lives elsewhere on the filesystem and is reached via a symlink inside the code repo.
5. **Other**: I'll describe.

For bank scopes broader than a single repo (team, domain, org), a separate-repo option is usually the right fit. Colocation makes the bank feel like "this project's notes," which reinforces the fragmentation problem Q1 tries to avoid.

---

### Question 4: Directory name

(Clickable.) "What should the memory bank directory be called?"

Options:

1. `memory-bank/` (default; conventional).
2. Custom (I'll tell you the name).

If custom, ask freeform: "What name?"

---

### Question 5: Roles and organization

(Clickable.) "Who will author records across the scope you named?"

Options:

1. Just me (solo contributor).
2. A single team (one shared governance).
3. Multiple roles with **distinct** governance (e.g., architects, PMs, developers, support; each owns their own records).
4. Multiple roles with **shared** ownership of domains or areas.

Based on the answer, recommend a physical organization (independent of scope; any of these can work at any scope):

- 1 or 2 → **Option C: single repo, folders by type** (`decisions/`, `rules/`, `exceptions/`, `context/`).
- 3 → **Option A: per-role** (`architecture-memory-bank/`, `product-memory-bank/`, etc., each with record-type folders).
- 4 → **Option B: per-domain** (`commerce-memory-bank/`, `platform-memory-bank/`, etc.).

State the recommendation briefly and ask (clickable): "Go with the recommendation, or pick a different layout?"

Options:

1. Go with the recommendation.
2. Option A (per-role).
3. Option B (per-domain).
4. Option C (single repo, folders by type).

Then, for any named role that matches a persona hydration guide, point the user at the corresponding file:

- Architects → `guide/by-persona/architects.md`
- Product managers / product owners → `guide/by-persona/pms.md`
- Developers → `guide/by-persona/developers.md`

Roles without a matching persona guide work from the spec files directly; the base schema applies to every role.

---

### Question 6: Exception tracking

(Clickable.) "Do you want to track exceptions as first-class records? Exceptions capture sanctioned deviations from rules: 'we broke the rule intentionally, here's why, here's who approved it.'

I strongly recommend YES. Teams that skip exception tracking accumulate invisible holes in their standards: informal permissions that live in Slack threads, can't be audited, and silently erode governance over time. The one case where it might be reasonable to skip: you're a solo developer with no standing rules yet."

Options:

1. Yes (recommended).
2. No (I don't have rules yet, or I'll fold exceptions into Decisions).

Based on answer:

- Yes → scaffold 4 record directories (decisions/, rules/, exceptions/, context/) and include Exception content in the instructions file.
- No → scaffold 3 record directories (decisions/, rules/, context/) and remove Exception content from the instructions file.

---

### Question 7: Scope vocabulary (optional)

(Clickable.) "Do you want to capture the canonical scope vocabulary for this bank now? This is the list of terms used to scope records: service names, domain names, customer segments, system IDs. Seeding it helps agents match queries to records from day one. Skipping is fine; you can always add it later.

If the bank spans multiple teams, the vocabulary should cover the full scope, not just one team's terms. Agents filtering on one team's terminology miss records tagged with another's. Agree on shared vocabulary across contributors before populating the bank."

Options:

1. Skip for now.
2. Capture now.

If option 2, ask freeform: "List 3–10 canonical scope terms for the full bank (comma-separated is fine). Examples: service names like `order-service`, domain names like `commerce`, segments like `tier-1`, system IDs like `plant-03`."

---

### Question 8: Agent integration

(Clickable.) "Which AI coding assistant will primarily consume this memory bank? This determines the name and location of the generated instructions file."

Options:

1. GitHub Copilot (deploys to `.github/copilot-instructions.md`).
2. Claude Code (deploys to `CLAUDE.md`).
3. Cursor (deploys to `.cursorrules`).
4. Other / multiple (I'll deploy to `.github/copilot-instructions.md` as a starting point; you adapt).
5. Not sure yet (I'll generate a generic version you can adapt).

---

### Now scaffold the memory bank.

Using the answers above, create the following.

#### 1. Directory structure

Create directories in the target repo (or separate memory bank repo per Q3) based on Q4 (directory name), Q5 (organization), and Q6 (exception tracking). A typical single-team colocated setup with exception tracking produces:

```
memory-bank/
├── decisions/
├── rules/
├── exceptions/
├── context/
└── .gitkeep (in each empty record directory)
```

Step 2 below adds `_templates/` alongside the record directories. Step 3 deploys an agent instructions file at the location determined by Q8 (typically at the root of the code repo, not inside `memory-bank/`).

#### 2. Template files

Create one template file per type in a `_templates/` directory inside the memory bank. Skip `exception.md` if Q6 is no.

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

**`_templates/exception.md`** (skip if Q6 is no)

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

<what prevents compliance, why the constraint exists, what mitigates risk>

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

#### 3. Agent Instructions File

Based on Q8, customize and deploy the memory bank starter as the team's instructions file.

**Starter:** `memory-bank/copilot-instructions.starter.md` (in this reference repo). Do not deploy as-is; it contains `<!-- SCAFFOLD:TOKEN -->` markers flagging spots that vary by bank.

**Deployment target (based on Q8):**

- GitHub Copilot → `.github/copilot-instructions.md` in the code repo
- Claude Code → `CLAUDE.md` in the code repo root
- Cursor → `.cursorrules` in the code repo root
- Other / not sure → start from `.github/copilot-instructions.md`; user adapts for their tool

**Replacement guide.** For each marker in the starter, apply the rule below based on Q answers, then strip the marker comment. After all markers are resolved, strip the top-of-file starter banner too. The deployed file should be clean operational content; the agent reads it every session, so every line should earn its keep.

| Marker                               | Replacement rule                                                                                                                                                                                                                                                                                                                                                                |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SCAFFOLD:RECORDS-LAYOUT-START/END`  | Rewrite the wrapped record-directory list to match Q5 (physical organization), Q3 (memory bank location), and Q4 (directory name). Default assumes colocated `memory-bank/` with per-type subdirs. Adjust paths if the memory bank lives elsewhere (separate repo local, separate repo remote via `gh`, symlinked, custom directory name) or uses per-role / per-domain layout. |
| `SCAFFOLD:EXCEPTION-START/END`       | Keep the wrapped content if Q6 answered yes. Remove the wrapped content if no. Multiple pairs throughout the file; apply consistently.                                                                                                                                                                                                                                          |
| `SCAFFOLD:SCOPE-VOCABULARY-HOOK`     | If Q7 captured scope vocabulary, insert a new `## Scope Vocabulary` section at the hook listing those terms. Otherwise, remove the marker without inserting anything.                                                                                                                                                                                                           |
| `SCAFFOLD:PERSONA-GUIDES-START/END`  | Keep only the persona links for roles captured in Q5. Adjust path prefixes if the memory bank lives somewhere other than `memory-bank/` in the target repo.                                                                                                                                                                                                                     |
| `SCAFFOLD:GUIDE-PATH/TEMPLATES-PATH` | Adjust path prefixes alongside `RECORDS-LAYOUT` based on memory bank location. If templates aren't created, remove the `TEMPLATES-PATH`-wrapped sentence.                                                                                                                                                                                                                       |

After replacements, every `<!-- SCAFFOLD:... -->` comment should be gone. Write the result to the deployment target chosen above.

#### 4. Namespace convention

(Clickable.) "What namespace prefix should your records use? Appears in IDs like `<namespace>-ADR-0001`."

Options:

1. Project name (e.g., `commerce`).
2. Team name (e.g., `platform-team`).
3. Domain name (e.g., `fulfillment`).
4. Short custom string (I'll provide).

If custom, ask freeform: "What namespace?"

Pick a namespace that matches the bank's scope (Q1). A domain-scoped bank with a project-name namespace will confuse future authors about what the bank actually covers.

Use the answer to pre-fill the templates.

#### 5. Seed records (optional)

(Clickable.) "Want to write 1–2 starter records now, or start with empty templates?"

Options:

1. Write 1–2 seed records now (I'll walk you through).
2. Start with empty templates; I'll write my own later.

If option 1, walk through 1–2 records using the templates, filling in real content from the user's answers. If the user's role matches a persona hydration guide in `guide/by-persona/`, read that guide first and use its field-fill cheat sheet and worked examples to shape phrasing. Templates give the structure; the persona guide gives the voice.

---

### After scaffolding, summarize what was created

- List all directories and files created.
- Remind the user of the four-stage retrieval funnel.
- Point them at `model/` docs for deeper reference and `guide/` for hydration practice.
- Remind them of the scope they chose (Q1) and the governance implications; if scope is broad, suggest a brief conversation with contributors about how records get proposed and accepted across teams.
- Suggest the next step: "Write your first real record. Start with a Decision; they're the most familiar and immediately valuable."
````
