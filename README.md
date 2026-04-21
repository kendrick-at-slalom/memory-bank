# Memory Bank

A structured knowledge layer for AI-assisted development. Four record types, a shared base schema, a retrieval model, and a small relationship vocabulary — designed so that AI coding assistants can find and reason over organizational knowledge at generation time.

## What This Is

A memory bank is a collection of Markdown files with YAML frontmatter that captures the decisions, rules, exceptions, and environmental facts that shape how a team builds software. The structure exists to serve AI retrieval: agents filter on frontmatter fields during a cheap scope pass, then read only the bodies of records that survive filtering.

The model is deliberately lean. Required fields are the smallest set that makes a record useful. Adoption matters more than completeness.

## Repo Layout

```
./
├── README.md          # this file
├── CLAUDE.md          # agent instructions for working in this repo
├── SCAFFOLD.md        # executable prompt for setting up a memory bank
└── model/             # the full architecture specification
    ├── README.md      # architecture overview — types, retrieval, relationships, examples
    ├── 00-retrieval-model.md
    ├── 01-base-memory-record.md
    ├── 02-decision.md
    ├── 03-policy-rule.md
    ├── 04-exception.md
    ├── 05-context.md
    └── 06-relationships.md
```

## Getting Started

Run the scaffold prompt in [SCAFFOLD.md](SCAFFOLD.md) with an AI coding assistant (Claude Code, Copilot, Cursor, etc.) to interactively set up a memory bank for your project. The prompt asks a few questions about your context and then generates the directory structure, templates, and agent instructions.

For the full architecture — types, retrieval model, relationships, and worked examples — see [model/README.md](model/README.md).

## License

MIT
