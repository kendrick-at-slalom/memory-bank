# Retrieval Model

## Why This Document Exists

Everything downstream of the memory bank schema (what fields to include, which ones to require, how to organize records physically) depends on a single question: **how will an agent actually find a record when someone asks a question?**[^rag]

If you don't have a clear answer to that, the schema design is arbitrary. If you do, the fields stop looking decorative and start looking like the interface they are.

This document describes the retrieval model in four layers:

1. Physical organization
2. Frontmatter filtering
3. The four-stage retrieval funnel
4. Agent plumbing

---

## The Core Idea

Retrieval happens in two passes:

1. **Scope pass.** The agent narrows the universe of records to the ones that could possibly be relevant, using cheap structured filters. No body text is read yet.
2. **Content pass.** The agent retrieves the full body of the handful of records that survived the scope pass, reads them, and synthesizes an answer.

Semantic search (embeddings, vector similarity) is part of the content pass, not the scope pass. The scope pass is pure structured filtering[^pushdown]:

1. Read the frontmatter header of each candidate file
2. Match against the query's scoping criteria
3. Discard anything that doesn't fit

Only then does the agent spend expensive tokens reading bodies.

This matters because the scope pass determines whether retrieval is _usable at scale_. A memory bank with a thousand records and no structured filtering means the agent is semantically searching a thousand bodies on every query: slow, expensive, and noisy. The same memory bank with good frontmatter filtering narrows to five to ten candidates in milliseconds and only reads those bodies. The difference is three orders of magnitude in cost and a much better answer.

---

## Layer 1: Physical Organization

The first-level filter is _where to look_. Before any frontmatter field is read, the agent has to decide which repository or directory to query.

Three options:

### Option A: One Repo Per Role or Function

```
product-memory-bank/
architecture-memory-bank/
design-memory-bank/
support-memory-bank/
developer-memory-bank/
```

Each role's records live in their own repository with their own governance. Cross-role queries hit multiple repos; within-role queries hit one.

**Pros**

- Hard scoping by default. PMs can't accidentally touch architecture records.
- Governance is clean: each repo has its own review process, owners, and permissions.
- Retrieval is obviously scoped: product questions go to the product repo.

**Cons**

- Cross-role queries are slightly harder (multiple repos to search).
- Records that span roles have no obvious home and might get duplicated.

**Best for:** Federated teams, distinct governance per role, no existing central knowledge tooling.

### Option B: One Repo Per Domain, Role as a Field

```
commerce-memory-bank/
fulfillment-memory-bank/
platform-memory-bank/
```

All roles write into the same domain repo. The `owners.role` field distinguishes authorship.

**Pros**

- Everything about a domain is in one place, regardless of role.
- Cross-role awareness: PMs see architecture records and vice versa.
- Maps naturally to domain-driven organization.

**Cons**

- Governance is complicated: one repo has multiple review processes.
- Permissions are harder to manage.

**Best for:** Mature practices, strong domain ownership, existing culture of cross-functional collaboration.

### Option C: One Central Repo, Organized by Folders

```
memory-bank/
  decisions/
  rules/
  exceptions/
  context/
```

Everyone writes into the same place. Folder structure encodes type.

**Pros**

- Maximally simple mental model. One place to look.
- Cross-cutting search is trivial.

**Cons**

- Governance and permissions are hardest at scale.
- Every change touches a shared repo.

**Best for:** Small teams, single products, individual developers, or as an end-state after consolidation.

### Choosing

Start with Option C unless you have a specific reason not to. It's the lowest-friction way to get started. Move to Option A or B when governance demands it; it's easier to split later than to untangle prematurely.[^conway]

---

## Layer 2: Frontmatter Filtering

Once the agent knows where to query, it narrows further using structured fields in the frontmatter. Four fields do most of the work:

### `owners`: Who Produced the Record

Filter: _"Who authored this, by role?"_

Useful when a repo has mixed content from multiple roles. Less useful in per-role repos where every record shares the same author role.

### `applies_to`: What the Record Is About

Filter: _"Does this record apply to the thing I'm asking about?"_

`applies_to` is a structured map with keys for the dimensions that matter: `services`, `domains`, `products`, `systems`, etc. An agent working on `order-service` filters for records where `applies_to.services` includes `order-service`.

This is a faceted classification scheme: each key in the map is an independent dimension that can be queried in any combination without committing to a single hierarchy.[^ranganathan] The question "what are the right keys for `applies_to`?" is a facet design question: pick the dimensions your team actually queries on. Common starting facets are `services`, `domains`, and `systems`, but the right set depends on what your agents will be asked. A team that frequently asks "what applies to the mobile platform?" needs a `platforms` facet; one that never asks that question doesn't.

**This is the single highest-value filter field.** A record without `applies_to` is effectively invisible to scoped retrieval; the agent has no way to know if it's relevant without reading the whole body.

### `memory_type`: What Kind of Record It Is

Filter: _"Am I looking for decisions, policies, context, or exceptions?"_

A cheap, exact filter that cuts the candidate set dramatically. Most queries want one or two types, not all four.

### `tags`: Loose Topical Labels

Filter: _"Anything tagged with 'onboarding' or 'authentication'?"_

The loosest filter. Works best as a fallback or complement to the structured fields above.

### The Combined Query

A realistic scope-pass query:

> Find records where `memory_type = Decision`, `applies_to.services` includes `order-service`, `status = accepted`, and `tags` include any of `[event-sourcing, persistence]`.

That's a cheap structured query against the frontmatter of files. It returns maybe five candidates. The agent then reads those five bodies and writes an answer.

**Zero semantic guessing at the scope pass.** The structure does the work.

---

## Layer 3: The Four-Stage Retrieval Funnel

This is the operational protocol an agent should follow when querying the memory bank. Each stage is cheaper than the next; never skip to a later stage when an earlier one would suffice.

### Stage 1: Glob on Filenames

Zero file content loaded. Filenames carry type (via subfolder), slug, and ID.

```
# All decisions
Glob memory-bank/decisions/*.md

# Decisions with "kafka" in the slug
Glob memory-bank/decisions/*kafka*.md

# Resolve a specific record by ID
Glob memory-bank/**/*ADR-0042*
```

### Stage 2: Grep on Frontmatter

When filename isn't enough, grep structured fields. Frontmatter is lines 1-15 of each file, so matching lines come back cheap.

```
# All current decisions
Grep "^status: accepted" memory-bank/decisions/

# Records tagged with compliance
Grep "tags:.*compliance" memory-bank/

# Records applying to a specific service
Grep "applies_to:.*order-service" memory-bank/
```

### Stage 3: Frontmatter-only Read

Read with a line limit (e.g., first 15 lines) to pull just the YAML header. Roughly 300 tokens per record. Only on candidates that survived stages 1 and 2.

### Stage 4: Full Body Read

Only for the handful (three to five) of records that actually matter for the question at hand. This is where the agent reads prose, synthesizes, and reasons.

### Anti-pattern

Reading every record in full to find matches. If you catch yourself doing this, stop and go back to stage 1.

---

## Layer 4: Telling the Agent How to Use All This

The agent doesn't know any of this automatically. It has to be told which repos to consult and what the conventions are. Suggested mechanisms, in increasing order of sophistication:

### Mechanism 1: Instructions Files

A `.github/copilot-instructions.md`, `CLAUDE.md`, `.cursorrules`, or equivalent file tells the agent how to use the memory bank.

Example:

> When asked about architecture decisions or technical standards, consult the memory bank. Filter by frontmatter fields: `memory_type`, `applies_to`, and `status` before reading record bodies. Follow the four-stage retrieval funnel (glob → grep → frontmatter read → full read). Always cite the `id` of any record you use.

This is the simplest, most effective starting mechanism. It's teachable, version-controlled, and cheap.

### Mechanism 2: Per-repo Instructions

Each memory bank directory gets its own instructions file describing what it contains and how to query it. Gives each owner control over how their records are described to the agent.

### Mechanism 3: Spaces or Workspaces

If your AI tool supports workspace-scoped contexts (Copilot Spaces, Cursor workspaces, etc.), create one per memory bank. Scoping is user-driven and explicit.

### Mechanism 4: Custom MCP Server

The most sophisticated option. A custom MCP server wraps the memory bank and exposes a tool like `search_memory_bank(memory_type, applies_to, query)` that does structured filtering before any semantic search. The filtering logic is enforced by the server, not left to the agent's interpretation.

**Recommendation:** Start with Mechanism 1. It gets 80% of the value with 10% of the effort. Move to Mechanism 4 when the memory bank grows large enough to need enforcement over convention.

---

## What the Agent Actually Does at Query Time

End-to-end flow when someone asks _"what have we decided about persistence for the order service?"_:

1. **Agent reads its instructions.** Instructions say architecture questions go to the memory bank.
2. **Agent scopes retrieval.** Identifies the right directory or repo.
3. **Agent runs the four-stage funnel.**
   - Stage 1: `Glob decisions/*order*` or `decisions/*persist*`
   - Stage 2: `Grep "applies_to:.*order-service" decisions/`
   - Stage 3: Read frontmatter of survivors
   - Stage 4: Read full body of the 2-3 matches
4. **Agent synthesizes an answer**, citing records by their human-readable `id`.

Four stages, and only stage 4 is expensive.

---

## Three Query Origins, Same Funnel

The retrieval model is role-neutral: the funnel doesn't care who asked. What changes by role is which filter fields matter most and which record types get retrieved. Three short vignettes to ground the abstraction:

### Architect, reviewing an intake request

An architect reads a new intake description and wants to know what constraints already apply to the scope before designing. The agent runs:

- Stage 1: `Glob memory-bank/decisions/*commerce*` and `memory-bank/rules/*commerce*`
- Stage 2: `Grep "applies_to:.*order-service" memory-bank/` and `Grep "memory_type: Exception" memory-bank/rules/`
- Stage 3: Frontmatter on the survivors
- Stage 4: Full bodies on two Decisions and one PolicyRule

The filters that do the work: `memory_type`, `applies_to.services`, `applies_to.domains`. The architect is scanning for constraints to respect, not for commitments or gotchas.

### Developer, before starting an implementation task

A developer has a story to build and wants to know what rules and decisions apply before writing code. The agent runs the same funnel:

- Stage 1: Glob the service's slug in decisions and rules
- Stage 2: Grep for `memory_type: PolicyRule` + `applies_to.services` containing the service
- Stage 3: Frontmatter read to see enforcement levels and statuses
- Stage 4: Full body on required PolicyRules and any Context records flagging gotchas

The filters that do the work: `memory_type: PolicyRule` (to see what's imperative), `enforcement: required` (to see what's non-negotiable), `applies_to.services`. The developer's primary question is "what must I follow, and what should I know."

### PM, deciding whether to accept an intake request

A PM is about to commit an intake request to the backlog and wants to know whether relevant decision context already exists. The agent runs:

- Stage 1: Glob the product area's slug across all four types
- Stage 2: Grep for `applies_to.segments` or `applies_to.products` matching the request
- Stage 3: Frontmatter on survivors, looking for existing commitments (Context tagged `commitment`), prior prioritization Decisions, and product principles
- Stage 4: Full body on the most relevant two or three records

The filters that do the work: `memory_type: Context` + `tags: commitment` (to surface promises), `memory_type: Decision` + `applies_to.products` (to surface prior prioritization calls), `memory_type: PolicyRule` (to surface product principles). The PM's primary question is "what decision context exists already, and am I about to re-litigate it."

Same funnel, three entry points. The schema's structured fields are what make the same pattern serve three different role-shaped questions.

---

## The Key Insight

> **Structured fields are the interface to the agent, not decorative metadata.**

Every field someone skips ("I don't need to fill in `applies_to`, it's obvious from the body!") is a filter the agent can't use. A record without `applies_to` is invisible to scoped retrieval: the agent either retrieves it for every query (noisy) or for none (dead). The author thinks they saved thirty seconds of typing. What they actually did was make the record unfindable.

When you write a record, the frontmatter is for the agent and the body is for the human. The agent never reads your body unless the frontmatter tells it to. A well-written body with thin frontmatter is worse than a rough body with complete frontmatter, because the rough-body record will actually get found.

---

## Notes

[^rag]: The two-pass retrieval model draws on Retrieval-Augmented Generation (RAG) architecture, where an external knowledge base is consulted before generation. See Lewis, P., et al. (2020). ["Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks."](https://arxiv.org/abs/2005.11401) _NeurIPS 2020_. The memory bank's scope pass is a deterministic, structured-metadata version of the retrieval step in RAG: cheaper and more reliable than embedding-based filtering when metadata is well-maintained.

[^pushdown]: Cheap filtering before expensive operations is standard practice in database query optimization, known as _predicate pushdown_: push filter conditions as close to the data source as possible to minimize the rows that flow through expensive joins. See [Airbyte: Demystifying Predicate Pushdown](https://airbyte.com/data-engineering-resources/predicate-pushdown). The memory bank applies the same idea to document retrieval: frontmatter filtering is the pushdown; full body reading is the join.

[^conway]: The choice of physical organization mirrors Conway's Law: "Organizations which design systems are constrained to produce designs which are copies of the communication structures of these organizations." Conway, M. (1968). ["How Do Committees Invent?"](https://www.melconway.com/Home/Committees_Paper.html) _Datamation_. See also [Fowler, M. "Conway's Law."](https://martinfowler.com/bliki/ConwaysLaw.html) The recommendation to start unified and split later is effectively the _inverse Conway maneuver_: letting knowledge structure drive team structure rather than the reverse.

[^ranganathan]: `applies_to` is a faceted classification scheme. The idea of classifying items along multiple independent dimensions rather than a single hierarchy goes back to S.R. Ranganathan's Colon Classification (1933). Ranganathan's PMEST model (Personality, Matter, Energy, Space, Time) showed that the right facets depend on what queries users will run. See [Wikipedia: Faceted Classification](https://en.wikipedia.org/wiki/Faceted_classification); [Britannica: Colon Classification](https://www.britannica.com/science/Colon-Classification). For modern application to information architecture, see [UXmatters: Faceted Metadata for Information Architecture and Search](https://www.uxmatters.com/mt/archives/2006/06/faceted-metadata-for-information-architecture-and-search.php).
