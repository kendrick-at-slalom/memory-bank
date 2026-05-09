---
name: hydrate-propose
description: >
  Surface reconciled drafts for human review as a PR against the memory bank
  repo. Each draft includes source provenance, reconcile annotations, and
  suggested reviewers. Auto-merge is not allowed; status moves from proposed
  to accepted only via human approval.
---

# hydrate-propose

Input: annotated drafts from `hydrate-reconcile`.

When this skill is activated:

1. Group drafts into a coherent PR set. Default: one PR per source (e.g., all drafts extracted from one ARB transcript ship in one PR). Avoid mixing sources unless drafts cross-reference.
2. Stage each draft as a file in the appropriate directory of the bank repo:
   - Decisions → `decisions/`
   - PolicyRules → `rules/`
   - Exceptions → `exceptions/`
   - Context → `context/`
3. Compose a PR description that includes:
   - One-line summary per draft (use the `title` field)
   - Reconcile annotations (net-new, supersedes-X, vocabulary notes)
   - Source provenance (links back to the ADR file, transcript, PR, or other source)
   - Suggested reviewers based on each draft's `owners` field and the bank's CODEOWNERS, if present
4. Open the PR with the `proposed-records` label (or whatever convention the bank uses to flag in-review records).

## Output

A PR URL and the list of files staged.

## What this skill does NOT do

- It does not merge the PR. Acceptance is human-gated.
- It does not change the `status` field. Drafts stay `proposed` until merged AND the team's accepted-status convention is applied (some teams flip on merge, some require a follow-up commit).
- It does not auto-resolve reconcile conflicts. If `hydrate-reconcile` flagged a vocabulary mismatch, this skill propagates the flag; a human resolves it.

## Common gotchas

- Bundling unrelated drafts into one PR. Reviewers stall, and the whole PR sits.
- Skipping source links in the description. Provenance is what reviewers use to trust the draft.
- Forgetting CODEOWNERS-aware reviewer suggestions. Default Copilot reviewers may not match the bank's governance.
