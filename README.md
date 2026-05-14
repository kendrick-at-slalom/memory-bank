# Memory Bank

A structured knowledge layer for AI-assisted development. Four record types, a shared base schema, a retrieval model, and a small relationship vocabulary, designed so that AI coding assistants can find and reason over organizational knowledge.

## What This Is

A memory bank is a collection of **Markdown files with YAML frontmatter** that captures the decisions, rules, exceptions, and environmental facts that shape how a team builds software. The structure exists to support AI retrieval: agents filter on frontmatter fields during a cheap scope pass, then read only the bodies of records that survive the filter.

The model is deliberately lean; required fields are the smallest set that makes a record useful. **Adoption matters more than completeness.**

## Repo Layout

```
./
├── README.md                  # this file
├── CLAUDE.md                  # agent instructions for working in this repo (cross-tool root instructions)
├── SCAFFOLD.md                # universal copy-paste form of the scaffold prompt
├── copilot-instructions.starter.md    # template; SCAFFOLD customizes and deploys to `.github/copilot-instructions.md` in adopting repos
├── .claude/                   # Claude-canonical surface (also read by VS Code Copilot)
│   ├── agents/hydrator.md       # hydration-pipeline coordinator
│   ├── commands/                # Claude-format slash commands
│   └── skills/                  # reference SKILL.md files (cross-tool)
│       └── hydrate-{discover,extract,draft,reconcile,propose,verify}/
├── .github/                   # Copilot-canonical surface
│   ├── copilot-instructions.md  # thin pointer to CLAUDE.md
│   ├── prompts/                 # Copilot-format prompt files (`/scaffold-memory-bank`, etc.)
│   └── instructions/            # Copilot path-scoped instructions (applyTo globs)
├── model/                     # the full architecture specification
│   ├── README.md              # architecture overview: types, retrieval, relationships, examples
│   ├── CLAUDE.md              # path-scoped guidance for editing the spec
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
│   ├── ai-assisted-hydration.md  # six-phase pipeline for AI-assisted authorship
│   └── by-persona/            # role-specific hydration guides (architects, PMs, developers)
└── examples/                  # sample records across all four types
    ├── CLAUDE.md              # path-scoped guidance for examples discipline
    └── hydration-demo/        # synthesized source corpus for demoing AI-assisted hydration
```

See the "Agent surface" section in [`CLAUDE.md`](CLAUDE.md) for how the `.claude/` and `.github/` trees support Copilot and Claude Code with equivalent behavior — single canonical files where both tools natively read the same location, paired files where formats differ.

## Getting Started

Run the scaffold prompt with an AI coding assistant. If you use Copilot or Claude Code, type `/scaffold-memory-bank` directly in chat. For any other tool, copy-paste from [SCAFFOLD.md](SCAFFOLD.md). The prompt asks a few questions about your context and generates the directory structure, templates, and agent instructions.

For the practitioner-facing hydration guide (what to do first, how to write records in your role, how to verify your agent can find them), see [guide/README.md](guide/README.md).

For AI-assisted hydration (mining ADRs, transcripts, PRs, and other artifacts to surface candidate records), see [guide/ai-assisted-hydration.md](guide/ai-assisted-hydration.md). Reference skill files for each phase live in [.claude/skills/](.claude/skills/).

For the full architecture (types, retrieval model, relationships, and worked examples) see [model/README.md](model/README.md).

## License

MIT
