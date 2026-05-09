---
name: hydrate-extract
description: >
  Pull candidate findings from a single source location. Each finding is a
  one-sentence statement plus a pointer back to the source. Use deterministic
  parsing for structured sources (ADRs, commits) and AI semantic analysis for
  prose (transcripts, PRDs).
---

# hydrate-extract

Input: one source location from `hydrate-discover` output.

When this skill is activated:

1. Determine extraction mode for the source:
   - **Deterministic:** ADRs folder, commits, GitHub issues via `gh issue list`, ADO work items via `az boards work-item show`
   - **AI semantic:** meeting transcripts, free-form PR descriptions, PRDs, prose docs
2. For each detected unit (one ADR file, one PR description, one transcript section):
   - Identify the candidate type: Decision, PolicyRule, Exception, or Context (best guess; confirmation comes in `hydrate-draft`)
   - Pull a one-sentence statement of the choice/fact/rule/deviation
   - Note `source_refs` data: file path, line range, commit hash, ticket ID
   - Note any obvious scope signals (service names, domain names, segment names) for `applies_to` later
3. Output one finding per candidate as a small structured record.

Findings are not draft records yet. Resist filling full frontmatter at this stage; phases 3 and 4 do that work.

## Output shape

```yaml
findings:
  - source: docs/adrs/0042-event-sourcing-orders.md
    candidate_type: Decision
    statement: "Order domain adopted event sourcing using Kafka as the event log."
    scope_signals: [order-service, fulfillment-service, commerce]
    confidence: high
  - source: team-notes/transcripts/2026-04-15-arb.md
    candidate_type: Context
    statement: "Plant floor network only supports TCP, not UDP."
    scope_signals: [ot, plant-03]
    confidence: medium
```

## Common gotchas

- Long ADRs may contain multiple decisions. Split them into separate candidates.
- "Decision" language in a transcript often signals what was *under discussion*, not what was *decided*. Mark these `confidence: low` so reconcile and propose handle them carefully.
- Don't infer scope signals beyond what the source explicitly states; phase 3 has more context to fill `applies_to`.
