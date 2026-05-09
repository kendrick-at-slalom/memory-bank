---
name: hydrate-draft
description: >
  Shape extract findings into full memory bank record drafts. Maps each
  candidate to one of the four record types (Decision, PolicyRule, Exception,
  Context), fills frontmatter from the base + type-specific schemas, and writes
  a concise prose body. Drafts include source provenance and stay in
  `status: proposed`.
---

# hydrate-draft

Input: findings from `hydrate-extract`.

When this skill is activated:

1. For each finding, confirm the candidate type using the type-distinction rules:
   - **Decision** = a specific choice at a point in time
   - **PolicyRule** = standing guidance for many future choices
   - **Exception** = a sanctioned deviation from a PolicyRule
   - **Context** = an environmental fact other records depend on
2. Fill the base frontmatter from `{BANK_PATH}/model/01-base-memory-record.md`:
   - Generate a UUID v4
   - Set `status: proposed` (always; promotion to `accepted` is a human call)
   - Compose `id` using the team's namespace and type prefix (ADR, POL, EXC, CTX)
   - Pull `applies_to` from the finding's `scope_signals`; flag for review if signals are sparse
   - Populate `source_refs` from the finding's source data
3. Fill the type-specific fields:
   - Decision: `decision_question`, `decision_outcome`, `alternatives_considered`, `decision_drivers`
   - PolicyRule: `rule_statement`, `enforcement`, `rationale`, `scope_of_application`
   - Exception: `exception_to` (UUID of the policy being excepted), `justification`, `compensating_controls`, `review_by`
   - Context: `fact_statement`, `verifiability`, `assumptions`, `constraints`
4. Write a short prose body following the type's body conventions. Keep it concise; the frontmatter does the load-bearing work.

## Output

One Markdown file per candidate, written to a staging directory (not directly into the bank).

## Common gotchas

- Phrasing a Decision as a statement instead of a question. `decision_question` is a question; rewrite "We need to use Kafka" as "Which event streaming technology should we use?"
- Filling `applies_to` from inference rather than evidence. If the source doesn't name a scope, leave `applies_to` thin and flag for human input rather than guessing.
- Drafting an Exception without a `review_by` date. Exceptions without review dates become permanent shadow rules.
- Bundling multiple decisions into one record. If `alternatives_considered` has seven entries or the title joins clauses with "and," split them.
