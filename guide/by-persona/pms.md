---
status: v0 draft
last_updated: 2026-04-23
---

# Hydrating the Memory Bank: Product Managers / Product Owners

PMs should start with Context, not Decisions. You produce facts about customers, segments, and commitments that architects and developers depend on at generation time. **Those facts are often the most invisible knowledge in the org**, living in people’s heads or scattered across slide decks. **Getting them into structured records is the single best thing a PM can do** for the memory bank.

## Which Types Matter Most

| Type       | What it looks like for you                                                      |
| ---------- | ------------------------------------------------------------------------------- |
| Context    | Customer facts, market segments, commitments, SLA terms, regulatory constraints |
| Decision   | Prioritization calls, scope boundaries, “we will / we won’t” choices            |
| PolicyRule | Product principles, acceptance standards, governance rules you own              |

Exceptions are less common for PMs but do come up (e.g., a customer-specific carve-out from a product principle).

## When to Write a Record

- You captured a customer commitment or market fact
- You made a **prioritization call** or set a **scope boundary**
- You confirmed a product principle or acceptance standard
- You established a fact about a segment, region, or customer tier that **constrains downstream work**

>The test: if an architect or developer would make a different choice without knowing what you know, that knowledge belongs in the bank.

## Going Deeper

The full authorship guide has the field-fill cheat sheet in PM/PO voice, a worked example per record type, and guidance on how your Context records feed into architect Decisions via `constrained_by`: [PM/PO Authorship Guide](../../model/by-persona/pm-authorship.md)

For typical search patterns, see [Retrieval: PM/PO Queries](../retrieval.md#pms--pos).

## Tips

- Your Context records are often the _inputs_ to someone else’s Decision. A customer commitment you capture today shows up as a `constrained_by` reference in an architect’s Decision tomorrow. _That’s the cross-role link working._
- Use consistent vocabulary in `applies_to` and `tags`. If you call a segment “enterprise” in one record and “ent” in another, queries that use one term will miss the other.
- Commitments with deadlines belong in Context records, not in task trackers. The memory bank captures the _fact_ of the commitment; the tracker manages the _work_ to fulfill it.
