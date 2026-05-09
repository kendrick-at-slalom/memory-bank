---
name: hydrate-verify
description: >
  After a proposed record is accepted and merged, confirm Copilot can find it
  by round-tripping the source question. Asks the question the record was
  meant to answer; the record should appear in citations.
---

# hydrate-verify

Input: a freshly accepted record (post-merge).

When this skill is activated:

1. Construct the source question. For a Decision, this is usually `decision_question` reframed for retrieval (e.g., "What did we decide about event sourcing for the order domain?"). For a Context, ask the fact directly. For a PolicyRule, ask "what rules apply to <scope>?". For an Exception, ask "are there exceptions against <policy>?".
2. Run the question against each verification surface:
   - **Copilot Spaces** (if the bank repo is in a Space): ask in the Space.
   - **IDE Copilot:** open the bank repo and ask the question with `copilot-instructions.md` in scope.
   - **Copilot CLI:** from the bank repo directory, `copilot ask "<question>"`.
3. For each surface, check whether the new record appears in the citations or response.
4. Report results.

## When the record doesn't surface

In order of likelihood:

- **`applies_to` vocabulary mismatch.** The record uses different scope terms than the question. Adjust the record's `applies_to` or rephrase the question.
- **`status` filter.** Some agent configurations filter `proposed` records out of retrieval. Confirm `status: accepted`.
- **Wrong directory or wrong Space scope.** The record landed somewhere the agent isn't looking.
- **Frontmatter is too thin.** Minimal frontmatter makes records invisible to the scope pass; see `memory-bank/guide/leading-practices.md#frontmatter-discipline`.

## Output

A short report: which surfaces found the record, which did not, and any recommended frontmatter or scoping adjustments.

## Common gotchas

- Verifying only on one surface. Different surfaces use the bank differently; a record that surfaces in IDE Copilot but not in Spaces usually has a Space configuration issue, not a record issue.
- Treating "Copilot mentioned the topic" as success. The criterion is whether the *specific record* appears in citations, not whether Copilot has general knowledge of the topic.
