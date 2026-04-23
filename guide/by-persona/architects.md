---
status: v0 draft
last_updated: 2026-04-23
---

# Hydrating the Memory Bank: Architects

Decisions are the natural entry point. Architects already think in ADRs, and the memory bank’s Decision type extends that format without much friction. All four record types show up in architect work (PolicyRule, Context for system facts, Exception), but Decisions are where adoption starts.

## Which Types Matter Most

| Type       | What it looks like for you                                                                                         |
| ---------- | ------------------------------------------------------------------------------------------------------------------ |
| Decision   | Design review outcomes, technology selections, migration sequencing, boundary choices                              |
| PolicyRule | Standing patterns codified for your scope (“all persistent services use Postgres”)                                 |
| Context    | System facts other roles need at generation time (air-gapped networks, canonical data sources, vendor constraints) |
| Exception  | Sanctioned deviations you’ve granted or documented, with review dates                                              |

## When to Write a Record

- You made an architectural choice under constraints
- You codified a standing pattern as the way things are done
- You granted a sanctioned deviation from a rule
- You learned or confirmed a system fact that others depend on
- You completed a design review

A rough test: if you’re explaining the _why_ of a system shape to someone who wasn’t in the room, you’re probably writing a Decision or Context. If you’re setting rules for generated code in your scope, that’s a PolicyRule. Carving out a deviation? Exception.

## Going Deeper

The full authorship guide has the field-fill cheat sheet (how you’d naturally phrase each field), a complete worked example per record type, and guidance on how your Decisions link to PM Context via `constrained_by`:

[Architect Authorship Guide](../../model/by-persona/architect-authorship.md)

For typical search patterns and how they map to the retrieval funnel, see [Retrieval: Architect Queries](../retrieval.md#architects).

## Tips

- Start with your most recent design review. The decision you made there is your first record.
- **`applies_to` matters more than any other field** for architects. If your record doesn’t name the services, domains, or systems it governs, nobody’s agent will find it.
- When you write a PolicyRule, think about who might need an Exception against it. If the answer is “probably someone, eventually,” make sure the PolicyRule’s `id` is something you can reference later.
- Decisions that reference PolicyRules via `constrained_by` are more useful than standalone Decisions. The link tells Copilot which rules shaped the choice.
