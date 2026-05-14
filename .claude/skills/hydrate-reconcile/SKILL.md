---
name: hydrate-reconcile
description: >
  Compare drafts against the existing memory bank state. Annotates each draft
  as net-new, duplicate, supersedes-existing, or vocabulary-mismatch. Drafts
  in conflict surface for human resolution before propose.
---

# hydrate-reconcile

Input: drafts from `hydrate-draft`.

When this skill is activated:

1. For each draft, query the existing bank using the four-stage retrieval funnel (see `{BANK_PATH}/guide/retrieval.md`):
   - Glob the type's directory (e.g., `decisions/*.md`) for similar `id` patterns
   - Grep frontmatter for matching `applies_to` values
   - Read frontmatter on the surviving candidates
2. For each draft, annotate one of:
   - **Net new.** No matching record found.
   - **Duplicate of `<existing-id>`.** Same content already exists. Recommend dropping the draft or merging notable details.
   - **Supersedes `<existing-id>`.** The draft conflicts with an existing accepted record. Recommend marking the existing one for supersession.
   - **Vocabulary mismatch.** The draft uses different scope terms than existing records (e.g., `enterprise` vs `ent`). Flag for terminology resolution.
3. For supersession candidates, suggest the two-record update: old record gets `status: superseded` + `superseded_by`; new record gets `supersedes`.
4. For vocabulary mismatches, suggest the canonical scope vocabulary used by the rest of the bank.

## Output

The original drafts plus an annotation block in each, naming the reconcile decision and any related record UUIDs.

## Common gotchas

- Treating "similar topic" as duplicate. Two Decisions about Kafka are not necessarily duplicates if their `applies_to` differs.
- Missing supersession because the existing record uses different vocabulary. Search by `applies_to` and by tag/title similarity, not exact match alone.
- Ignoring `status`. A draft that conflicts with a `superseded` record is not a conflict; it's confirming the supersession.
