---
status: v0 draft
last_updated: 2026-04-23
---

# Hydrating the Memory Bank: Developers

Library choices and implementation patterns that reach beyond a single PR are the entry point here. You’ll touch all four record types, but Decisions and Exceptions tend to come up most often, since you’re making tool choices and documenting workarounds against standards set by architects.

## Which Types Matter Most

| Type       | What it looks like for you                                                                     |
| ---------- | ---------------------------------------------------------------------------------------------- |
| Decision   | Library or tool selection, implementation pattern choices, “why we did it this way”            |
| Exception  | Workarounds accepted against an architect’s PolicyRule, with justification and review date     |
| PolicyRule | Team-level practices you’ve codified (testing standards, naming conventions, deployment rules) |
| Context    | Gotchas, vendor behavior quirks, integration facts, “things the next person needs to know”     |

## When to Write a Record

- You chose a library or tool that has reach beyond the immediate PR
- You codified a team practice (linting rules, test patterns, deploy process)
- You discovered a gotcha or vendor behavior that others will hit
- You accepted a workaround against a standard and got it sanctioned

>If the next person would benefit from not having to re-discover this, it belongs in the bank.

## Going Deeper

The full authorship guide covers the field-fill cheat sheet in developer voice, a worked example per record type, and how your Decisions relate to architect PolicyRules (e.g., a library choice `constrained_by` a platform wrapper rule): [Developer Authorship Guide](../../model/by-persona/developer-authorship.md)

For typical search patterns, see [Retrieval: Developer Queries](../retrieval.md#developers).

## Tips

- Before you implement, ask Copilot what rules and decisions apply to the scope you’re about to touch. If nothing comes back, either the bank is thin for that area or your scoping is off.
- When you write an Exception (a workaround against a standard), always fill `review_date`. Exceptions without review dates become permanent shadow rules that nobody revisits.
- Context records about gotchas and vendor quirks punch above their weight. They capture knowledge that otherwise lives only in the heads of people who got burned.
- Decisions that reference an architect’s PolicyRule via `constrained_by` are more useful than standalone Decisions. The link tells Copilot which rules shaped your choice.
