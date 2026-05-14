---
description: Discipline for editing the memory bank schema specification under model/.
applyTo: 'model/**'
---

# Working in model/

The `model/` tree is the canonical schema spec. Before modifying any file:

- **Read the file end-to-end.** Sections like "open questions" and "common mistakes" carry hard-won rationale that is easy to lose in edits. Don't drop them when restructuring.
- **Lean over comprehensive.** The model's core design principle is that adoption matters more than completeness. When in doubt about adding a field, type, or concept — don't. Let real observed need justify additions.
- **Schema changes mirror into the scaffold prompt.** Any change to the base record schema (`01-base-memory-record.md`) or a type-specific schema (`02-decision.md` through `05-context.md`) must be reflected in all three scaffold-prompt forms: [`SCAFFOLD.md`](../../SCAFFOLD.md) (universal), [`.github/prompts/scaffold-memory-bank.prompt.md`](../prompts/scaffold-memory-bank.prompt.md) (Copilot), [`.claude/commands/scaffold-memory-bank.md`](../../.claude/commands/scaffold-memory-bank.md) (Claude). The three are kept in sync.
- **Base schema is shared.** `01-base-memory-record.md` is what every type extends. Type-specific extensions go in their own files (`02-decision.md` etc.), not in the base. If a field belongs to two types but not all four, that's a signal to look harder at the design before adding it.
- **Reading order matters.** When pointing a new contributor at the spec, lead with `00-retrieval-model.md`. It explains the design forces that shaped everything else; the type files only make sense once retrieval is understood.

The Claude-format equivalent of this file is [`model/CLAUDE.md`](../../model/CLAUDE.md). Keep both in sync when changing this guidance.
