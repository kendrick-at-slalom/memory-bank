---
status: v0 draft
last_updated: 2026-04-23
companion_to: model/00-retrieval-model.md
---

# Searching the Memory Bank

You’ll **hydrate** the memory bank with records you write. You’ll also **find and read records** written by others. This page covers how retrieval works, what the funnel looks like across different surfaces, and what typical queries look like for each role.

For when and how to _write_ records, see your role’s hydration guide:

- [Architects](by-persona/architects.md)
- [Product managers / product owners](by-persona/pms.md)
- [Developers](by-persona/developers.md)

## How Retrieval Works

Filters run cheapest first. Agents walk down this funnel whether you notice it or not, and you’re doing the same thing by hand when you Glob or Grep a local copy.

1. **Filename filter.** Narrow by record type (via subfolder) and slug. No file content loaded.
2. **Frontmatter filter.** Narrow by `applies_to`, `status`, `tags`, and other YAML fields. Still no bodies loaded.
3. **Frontmatter-only read.** Pull just the YAML header of the survivors. Roughly 300 tokens per record.
4. **Full body read.** Only for the three-to-five records that actually matter.

The anti-pattern is reading every record in full to find matches. If you catch yourself doing that (or your agent is doing it), go back to step 1.

The full retrieval model and rationale are in [`model/00-retrieval-model.md`](../model/00-retrieval-model.md).

## Same Funnel, Multiple Surfaces

The stages don’t change, but the tool does. If you can write the right Glob/Grep command, you can phrase the right Copilot prompt.

| Stage                     | Bare filesystem / IDE          | Copilot (IDE or CLI)                                | Copilot Spaces                              |
| ------------------------- | ------------------------------ | --------------------------------------------------- | ------------------------------------------- |
| 1. Filter by type/scope   | `Glob decisions/*.md`          | Reads repo files, follows `copilot-instructions.md` | Space selection + `copilot-instructions.md` |
| 2. Filter on frontmatter  | `Grep "applies_to:.*commerce"` | Prompt phrasing that names the scope                | Prompt phrasing that names the scope        |
| 3. Read candidate headers | `Read` with `limit: 15`        | Reads file, returns summary                         | Copilot reads citations automatically       |
| 4. Read full bodies       | `Read` (no limit)              | Reads full file on request                          | Copilot expands citations on request        |

Copilot CLI and IDE Copilot both read `copilot-instructions.md`, so the steering is the same regardless of whether you work in the editor or the terminal. Copilot Spaces adds a scoping layer on top — it tells Copilot _which repos_ to look in, while `copilot-instructions.md` tells it _how_ to use what it finds.

## What Typical Queries Look Like, by Role

### Architects

You’re usually looking for Decisions that constrain what you’re designing, PolicyRules that apply to systems you own, or Context about dependencies you inherit.

- “What Decisions affect the commerce order service?” → `memory_type: Decision` + `applies_to` contains `commerce`
- “What PolicyRules apply to OT systems?” → `memory_type: PolicyRule` + `applies_to` contains `ot`
- “Are there sanctioned Exceptions against the mTLS policy?” → `memory_type: Exception` + `exception_to` references the mTLS rule’s `id`
- “Who owns the Oracle Price Management integration?” → `memory_type: Context` + tags for ownership; often a body search on the `fact_statement` field

If you review intake requests, the funnel is also the shape of your pre-read: glob for Decisions that apply to the requester’s scope, grep for any Exceptions, skim the headers, open the two or three that matter. You’re rehearsing the query your agent will run on its own when you hand it an intake description.

### Developers

You’re usually looking for PolicyRules that apply to the service you’re building, Decisions that explain why something is the way it is, or Exceptions that cover your specific case.

- “What PolicyRules apply to the service I’m working on?” → `memory_type: PolicyRule` + `applies_to` contains your service’s scope name
- “Why did we pick Postgres here?” → `memory_type: Decision` + `applies_to` contains your service; grep the `decision_question` or `decision_outcome` fields
- “Is there an Exception that covers my plant-03 telemetry service?” → `memory_type: Exception` + `applies_to` contains `plant-03`
- “What Context should I know about this integration?” → `memory_type: Context` + `applies_to` contains the dependency name

Before you implement, ask Copilot what rules and decisions apply to the scope you’re about to touch. If nothing comes back, either the bank is thin for that area or your scoping is off. Both are fixable.

### PMs / POs

You’re usually looking for Decisions that shape what’s possible for a feature, Context about customers or markets, or PolicyRules that constrain priorities.

- “What Decisions have we made about [customer segment / feature area]?” → `memory_type: Decision` + `applies_to` on the segment or area
- “What Context exists about our enterprise SLAs?” → `memory_type: Context` + `tags` includes `sla`
- “What PolicyRules constrain this feature idea?” → `memory_type: PolicyRule` + `applies_to` contains the relevant scope (regulatory, product area, customer tier)
- “What commitments have we made in [region]?” → `memory_type: Context` + tags for commitment; or Decisions explicitly framed as commitments, depending on how your governance handles that shape

If you’re deciding whether to accept an intake request, the funnel gives you a quick read on the decision context that’s already captured. A request with nothing to anchor to (no Context, no Decisions) is a signal to probe harder, not a signal to fast-track.

## See Also

- [Leading Practices](leading-practices.md): frontmatter discipline, lifecycle, cross-role work, anti-patterns, and how to verify your records are findable.
