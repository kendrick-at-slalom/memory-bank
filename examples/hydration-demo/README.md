# Hydration Demo Fixture Set

Synthesized source corpus for demoing the [AI-assisted hydration pipeline](../../guide/ai-assisted-hydration.md). Everything below is fictional; no real company is behind it.

## Layout

```
hydration-demo/
├── sources/                # raw input artifacts
│   ├── adrs/
│   │   ├── 0007-event-sourcing-orders.md
│   │   └── 0011-postgres-everywhere.md
│   ├── transcripts/
│   │   └── 2026-04-15-arb-snippet.md
│   └── prs/
│       └── PR-1308-redis-exception.md
└── expected/               # what AI should produce after the pipeline runs
    ├── decisions/commerce-ADR-0042.md
    ├── rules/platform-POL-0005.md
    ├── exceptions/platform-EXC-0003.md
    └── context/commerce-CTX-0019.md
```

## Demo flow

1. Start with the `sources/` tree. These are the kinds of artifacts a real team produces.
2. Run the hydration pipeline (or walk through it manually) against those sources.
3. Compare the AI-produced drafts against the records in `expected/`. The expected records are an answer key; AI output should be substantially similar in structure and content.

## What this demonstrates

Each source maps to a different record type:

| Source | Produces | Why |
| --- | --- | --- |
| `adrs/0007-event-sourcing-orders.md` | Decision | A specific architectural choice with alternatives considered |
| `adrs/0011-postgres-everywhere.md` | PolicyRule | A standing rule that applies to future services, not just current ones |
| `transcripts/2026-04-15-arb-snippet.md` | Context | A system fact established mid-discussion |
| `prs/PR-1308-redis-exception.md` | Exception | A sanctioned deviation from a PolicyRule |

The transcript is intentionally ambiguous: it also contains a decision under discussion (not yet decided) and an emerging consensus that could become a future PolicyRule. AI should mark these `confidence: low` and surface them for review rather than promoting them as Decisions or Rules.
