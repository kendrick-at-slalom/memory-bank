# Memory Bank

A structured knowledge layer for AI-assisted development. Four record types, a shared base schema, a retrieval model, and a small relationship vocabulary, designed so that AI coding assistants can find and reason over organizational knowledge.

## What This Is

A memory bank is a collection of **Markdown files with YAML frontmatter** that captures the decisions, rules, exceptions, and environmental facts that shape how a team builds software. The structure exists to support AI retrieval: agents filter on frontmatter fields during a cheap scope pass, then read only the bodies of records that survive the filter.

The model is deliberately lean; required fields are the smallest set that makes a record useful. **Adoption matters more than completeness.**

## Repo Layout

```
./
├── README.md                  # this file
├── CLAUDE.md                  # agent instructions for working in this repo
├── SCAFFOLD.md                # executable prompt for setting up a memory bank
├── copilot-instructions.starter.md    # template; SCAFFOLD customizes and deploys to `.github/copilot-instructions.md` in adopting repos
├── model/                     # the full architecture specification
│   ├── README.md              # architecture overview: types, retrieval, relationships, examples
│   ├── 00-retrieval-model.md
│   ├── 01-base-memory-record.md
│   ├── 02-decision.md
│   ├── 03-policy-rule.md
│   ├── 04-exception.md
│   ├── 05-context.md
│   └── 06-relationships.md
├── guide/                     # practitioner-facing hydration guide
│   ├── README.md              # what a memory bank is, how to get started
│   ├── retrieval.md           # how Copilot finds records, the retrieval funnel
│   ├── leading-practices.md   # cross-cutting discipline: frontmatter, lifecycle, verification
│   └── by-persona/            # role-specific hydration guides (architects, PMs, developers)
└── examples/                  # sample records across all four types
```

## Getting Started

Run the scaffold prompt in [SCAFFOLD.md](SCAFFOLD.md) with an AI coding assistant (Claude Code, Copilot, Cursor, etc.) to interactively set up a memory bank for your project. The prompt will ask a few questions about your context and then generate the directory structure, templates, and agent instructions.

For the practitioner-facing hydration guide (what to do first, how to write records in your role, how to verify Copilot can find them), see [guide/README.md](guide/README.md).

For the full architecture (types, retrieval model, relationships, and worked examples) see [model/README.md](model/README.md).

## License

MIT
