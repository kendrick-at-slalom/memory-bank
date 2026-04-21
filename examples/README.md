# Example Memory Bank

This directory contains a complete example memory bank for a fictional developer platform. It is intended to illustrate the architecture defined in the [`model/`](../model/) directory — specifically how the four record types work together, what different lifecycle states look like, and how relationships between records enable an agent to reason over connected knowledge.

The examples use a consistent domain: a SaaS developer platform with four service domains (platform, identity, builds, artifacts). All decisions, rules, exceptions, and context records are fictional.

---

## What's illustrated

### All four record types

| File                                                               | Type       | What it shows                                                                                  |
| ------------------------------------------------------------------ | ---------- | ---------------------------------------------------------------------------------------------- |
| [decisions/platform-ADR-0001.md](decisions/platform-ADR-0001.md)   | Decision   | Full field set: `alternatives_considered`, `decision_drivers`, `approved_by`                   |
| [decisions/platform-ADR-0002.md](decisions/platform-ADR-0002.md)   | Decision   | `constrained_by` relationship to a PolicyRule                                                  |
| [decisions/platform-ADR-0003.md](decisions/platform-ADR-0003.md)   | Decision   | `status: superseded` — the "old" half of a supersession pair                                   |
| [decisions/platform-ADR-0006.md](decisions/platform-ADR-0006.md)   | Decision   | `status: rejected` — considered and turned down, preserved for history                         |
| [decisions/platform-ADR-0007.md](decisions/platform-ADR-0007.md)   | Decision   | Decision that rests on an Exception; links to both the rule and the carve-out                  |
| [decisions/platform-ADR-0011.md](decisions/platform-ADR-0011.md)   | Decision   | `supersedes` ADR-0003 — the "new" half of a supersession pair                                  |
| [rules/platform-POL-0004.md](rules/platform-POL-0004.md)           | PolicyRule | `enforcement: required`; `derived_from` its originating Decision; `exceptions_allowed: true`   |
| [rules/platform-POL-0009.md](rules/platform-POL-0009.md)           | PolicyRule | `enforcement: required`; security-focused rule with higher Exception bar                       |
| [rules/platform-POL-0012.md](rules/platform-POL-0012.md)           | PolicyRule | `status: proposed` — under discussion, not yet an accepted constraint                          |
| [exceptions/platform-EXC-0002.md](exceptions/platform-EXC-0002.md) | Exception  | `exception_to` points at POL-0004; `compensating_controls`; `scope_boundary`; `review_by` date |
| [context/platform-CTX-0005.md](context/platform-CTX-0005.md)       | Context    | Environmental fact that justifies a downstream Exception                                       |
| [context/platform-CTX-0008.md](context/platform-CTX-0008.md)       | Context    | Time-bounded fact: `effective_to` set; expires when migration completes                        |

### All lifecycle states

| Status       | Example                                                                                                                                                                   |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `proposed`   | [POL-0012](rules/platform-POL-0012.md) — under ARB review, not yet authoritative                                                                                          |
| `accepted`   | [ADR-0001](decisions/platform-ADR-0001.md), [POL-0004](rules/platform-POL-0004.md), [EXC-0002](exceptions/platform-EXC-0002.md), [CTX-0005](context/platform-CTX-0005.md) |
| `superseded` | [ADR-0003](decisions/platform-ADR-0003.md) — replaced by ADR-0011; `superseded_by` field set                                                                              |
| `rejected`   | [ADR-0006](decisions/platform-ADR-0006.md) — considered and turned down by ARB; preserved with rationale                                                                  |

### All relationship types

| Relationship                   | Where it appears                                                                 |
| ------------------------------ | -------------------------------------------------------------------------------- |
| `supersedes` / `superseded_by` | ADR-0011 supersedes ADR-0003 (both records store their half of the link)         |
| `derived_from`                 | POL-0004 → ADR-0001 (the policy was codified from the decision)                  |
| `constrained_by`               | ADR-0002 → POL-0009; ADR-0007 → POL-0004 (the decision operates within the rule) |
| `depends_on`                   | EXC-0002 → CTX-0005 (the Exception assumes the SLA context is true)              |
| `relates_to`                   | ADR-0007 → EXC-0002 (the decision links to its authorizing Exception)            |

---

## The record graph

```
ADR-0001 ──derived_from──► POL-0004 ◄──constrained_by── ADR-0007
                               │                             │
                               │ (exception_to)              │ (relates_to)
                               ▼                             ▼
                           EXC-0002 ◄──depends_on── CTX-0005

ADR-0002 ──constrained_by──► POL-0009

ADR-0003 ──superseded_by──► ADR-0011
ADR-0011 ──supersedes──────► ADR-0003

ADR-0006 (rejected — no outbound links, references POL-0004 in body)
CTX-0008 (isolated — time-bounded operational fact, no record links needed)
POL-0012 (proposed — not yet linked to other records)
```

---

## The supersession pair (ADR-0003 / ADR-0011)

The supersession of ADR-0003 by ADR-0011 demonstrates the two-record operation described in [`06-relationships.md`](../model/06-relationships.md):

- **ADR-0003**: `status: superseded`, `superseded_by: f6a7b8c9-...` (ADR-0011's UUID), `effective_to: 2026-01-20`
- **ADR-0011**: `supersedes: [c3d4e5f6-...]` (ADR-0003's UUID), `effective_from: 2026-01-20`

An agent encountering ADR-0003 should resolve `superseded_by` to ADR-0011 before acting on the encryption standard. ADR-0003 is preserved for institutional memory (the SOC 2 audit trail specifically needed the history of when and why AES-128 was replaced).

---

## The exception chain (CTX-0005 → EXC-0002 → ADR-0007)

The relationship between the session SLA context, the PostgreSQL exception, and the Redis decision illustrates how Context records provide the environmental facts that justify Exceptions:

1. [CTX-0005](context/platform-CTX-0005.md) establishes the fact: 10ms p99 SLA on session validation.
2. [EXC-0002](exceptions/platform-EXC-0002.md) cites that fact (`depends_on` CTX-0005) to justify why Redis is needed despite POL-0004.
3. [ADR-0007](decisions/platform-ADR-0007.md) makes the architectural choice, operating under the authorization of EXC-0002.

If CTX-0005 ever changed (the SLA was relaxed to 25ms, for instance), the agent should surface EXC-0002 as a record that depends on it and may need revisiting.

---

## Retrieval examples

The four-stage retrieval funnel applies to this directory:

**Stage 1 — glob by type:**

```
examples/exceptions/**    →  narrows to EXC-0002
examples/rules/**         →  narrows to POL-0004, POL-0009, POL-0012
```

**Stage 2 — grep frontmatter for `status: accepted`:**

```bash
grep -l "status: accepted" examples/**/*.md
# returns everything except ADR-0003 (superseded), ADR-0006 (rejected), POL-0012 (proposed)
```

**Stage 3 — grep frontmatter for applies_to scope:**

```bash
grep -A5 "applies_to:" examples/rules/platform-POL-0004.md | grep "identity"
# confirms POL-0004 applies to the identity domain before reading the full body
```

**Stage 4 — full body read:**
Only for the 3–5 records that survive stages 1–3.

---

## What's not shown

These examples use Option C physical organization (single directory, type-based subfolders). Option A (per-role repos) and Option B (per-domain repos) are described in [`00-retrieval-model.md`](../model/00-retrieval-model.md) but not illustrated here.

The `_templates/` directory generated by the scaffold prompt is also not present — these examples pre-suppose a working knowledge of the schema. Use [SCAFFOLD.md](../SCAFFOLD.md) to generate templates for your own project.
