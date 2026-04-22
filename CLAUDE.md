# Claude Context — Memory Bank Reference Architecture

> **What this repo is:** A reference architecture for structured knowledge layers that AI coding assistants can query at generation time. It defines four record types, a shared base schema, a retrieval model, and a relationship vocabulary.
>
> **What this repo is NOT:** A template library, a pre-populated knowledge base, or a notes/capture pipeline. How records get *into* the memory bank (meeting notes, grooming workflows, promotion processes) is an implementation concern for the adopting team, not part of this architecture.

## Repo Layout

```
./
├── CLAUDE.md          # this file
├── README.md          # human-facing overview (start here for orientation)
├── SCAFFOLD.md        # executable prompt — walks an agent through setting up a memory bank
└── model/             # the full specification
    ├── README.md               # condensed overview of the model, points into the spec files
    ├── 00-retrieval-model.md   # WHY the schema is shaped this way
    ├── 01-base-memory-record.md # shared schema all types extend
    ├── 02-decision.md          # Decision type
    ├── 03-policy-rule.md       # PolicyRule type
    ├── 04-exception.md         # Exception type
    ├── 05-context.md           # Context type
    ├── 06-relationships.md     # how records link to each other
    └── by-persona/             # authorship guides per role — same schema, role-specific voice
        ├── README.md
        ├── architect-authorship.md
        ├── pm-authorship.md
        └── developer-authorship.md
```

**Reading order matters.** `00-retrieval-model.md` first — it explains the design forces that shaped everything else. Then `01-base-memory-record.md` for the shared schema. Type-specific files and relationships after that.

There is a second, parallel entry path through `model/by-persona/`. The persona guides don't add to the schema — they show each role what authorship looks like in their own voice, with worked records and authorship triggers. If the user identifies a specific role for themselves or their team, point them there as an on-ramp; they can read the spec files after. The persona guides and the schema files must stay consistent — changes to one may require changes to the other.

## How This Repo Gets Used

The primary use case: an agent is pointed at this repo and asked to help set up a memory bank for a target project. The workflow is:

1. Agent reads the model/ docs to understand the architecture.
2. Agent runs the SCAFFOLD.md prompt interactively with the user.
3. SCAFFOLD asks four questions (project context, roles, exception tracking, agent integration).
4. Agent generates directory structure, templates, and agent instructions in the target project.

The model/ docs are the reference spec. SCAFFOLD.md is the executable entry point. README.md is the human orientation layer.

## The Two Things That Matter Most

If an agent internalizes nothing else from this repo, it should be these:

### 1. The Four-Stage Retrieval Funnel

When querying a memory bank, always start cheap and narrow progressively:

| Stage | Operation | Cost | What it does |
|---|---|---|---|
| 1 | Glob on filenames | Zero content loaded | Narrow by type (subfolder), slug, date |
| 2 | Grep on frontmatter | Lines 1–15 per file | Filter structured fields without loading bodies |
| 3 | Frontmatter-only read | ~300 tokens per record | Pull just the YAML header of surviving candidates |
| 4 | Full body read | Full token cost | Only for the 3–5 records that actually matter |

**Anti-pattern:** Reading every record in full to find matches. If you catch yourself doing this, go back to stage 1. The funnel exists because memory banks grow — what works with 10 records must also work with 500.

### 2. The Status Lifecycle

Records move through a governed lifecycle. Status is not decorative — it determines whether an agent should treat a record as authoritative.

```
proposed → accepted → [superseded | deprecated | rejected]
```

- **`proposed`** — Under discussion. Not yet agreed. An agent should surface it as context but not treat it as a constraint.
- **`accepted`** — Current and authoritative. This is what the agent should act on.
- **`superseded`** — Replaced by a newer record. The `superseded_by` field points to the replacement. An agent should follow the link, not act on the stale record.
- **`deprecated`** — No longer applies. No replacement exists.
- **`rejected`** — Considered and explicitly declined. Kept for history.

**Supersession is a two-record operation.** When a record is superseded:
1. Old record: `status` → `superseded`, `superseded_by` → UUID of the new record.
2. New record: `supersedes` → [UUID of the old record].
3. Both records remain in the memory bank — history is preserved, not deleted.

An agent encountering a `superseded` record should always resolve the chain to find the current version before acting.

## How to Behave When Working in This Repo

- **This is a reference architecture, not an engagement artifact.** No client names, no engagement-specific content, no proprietary examples. Examples should be generic and illustrative.
- **The model/ docs have accumulated rationale.** Read a file end-to-end before modifying it. Sections like "open questions" and "common mistakes" carry hard-won reasoning. Don't lose them in edits.
- **Lean over comprehensive.** The model's core design principle is that adoption matters more than completeness. When in doubt about adding a field, type, or concept — don't. Let real observed need justify additions.
- **SCAFFOLD.md is executable.** Changes to the model should be reflected in SCAFFOLD.md's templates and instructions. The two must stay consistent.
- **README.md is the human entry point.** It should stay readable as a standalone overview. Deep detail belongs in the model/ docs, not in the README.
