---
name: hydrate-discover
description: >
  Inventory the source artifacts available for hydrating the memory bank.
  Surveys ADRs, PR descriptions, commit history, transcripts, PRDs, and other
  artifacts that produce candidate Decision/PolicyRule/Exception/Context
  records. Run once at setup and periodically thereafter.
---

# hydrate-discover

When this skill is activated:

1. Read `.github/copilot-instructions.md` for any declared source pointers (e.g., "ADRs live in `docs/adrs/`", "transcripts archived in `team-notes/transcripts/`").
2. If no declared pointers, scan the repo for likely sources:
   - `docs/adrs/`, `docs/decisions/`, or any folder containing files matching `NNNN-*.md`
   - `README.md` and other docs for links to scope docs, PRDs, or wiki pages
   - Recent git history (last 90 days): `git log --since="90 days ago" --pretty=format:"%h %s"`
   - `.github/PULL_REQUEST_TEMPLATE.md` to understand the team's PR description shape
3. Note external sources the agent cannot reach but should know about (Slack, ADO work items, SharePoint, Confluence). Mark these as "out of scope for automated extraction" rather than skipping silently.

## Output

A Markdown table with columns: source, location, type, automation level (deterministic / AI-semantic / out-of-scope).

This skill does not extract content; it only inventories. Pass the output to `hydrate-extract` for actual candidate findings.

## Common gotchas

- ADRs in folders without consistent naming need manual flagging
- Transcripts in private channels or DMs are out-of-scope
- A repo with no `docs/adrs/` may still capture decisions in PR descriptions; don't skip the PR pass just because the obvious folder is missing
