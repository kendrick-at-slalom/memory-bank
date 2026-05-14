---
status: v0 draft
last_updated: 2026-05-08
---

# AI-Assisted Hydration

Most of this guide assumes a human notices a moment that should produce a record and writes it. AI can do more than verify retrieval at the end; it can also surface candidate records from sources you already have. This page covers the pipeline for that work.

The shape is six phases, run as a sequence of skills or as one combined agent depending on tooling. Each phase has a clear input and output, and human review sits at the propose step before any record lands in the bank.

## Where AI Helps, and Where It Doesn't

AI is good at:

- Scanning heterogeneous sources (ADRs, PRs, commit messages, meeting transcripts, PRDs) for candidate records
- Drafting frontmatter that matches the schema: filling `applies_to`, suggesting tags, framing `decision_question`
- Detecting duplicates against existing records and flagging supersession candidates
- Round-tripping a new record by asking the question it was meant to answer

AI should not:

- Auto-merge candidates into the bank. Promotion to `accepted` is a human call.
- Decide a record's `memory_type` without confirmation. Suggestion is fine; the final type is an architect or PO call.
- Resolve conflicts between candidates and existing records on its own. Conflicts surface for review.

The friction is the feature. The bank earns trust from human review at the gate; agent automation accelerates getting candidates to the gate, not crossing it.

## The Pipeline

Six phases, each invokable on its own or chained. The last phase verifies retrieval; the others move candidates from a source to a proposed record.

| Phase | Input | Output |
| --- | --- | --- |
| **1. discover** | Repo state, source pointers in `copilot-instructions.md` | Inventory of source locations: ADRs folder, transcripts directory, PR descriptions, etc. |
| **2. extract** | A specific source location | Raw candidate findings, one per detected decision/rule/fact/exception |
| **3. draft** | Raw candidates | Filled record drafts: frontmatter + body + provenance back to source |
| **4. reconcile** | Drafts + existing bank state | Annotated drafts: net-new, duplicate of, supersedes, vocabulary mismatch |
| **5. propose** | Reconciled drafts | PR or staged change with `status: proposed` |
| **6. verify** | A newly accepted record | Round-trip test: did Copilot find the record when asked the source question? |

Reference skills for each phase live in [`.claude/skills/`](../.claude/skills/). Each is a single `SKILL.md` invocable as a slash command. They are read natively by both Claude Code and VS Code Copilot[^cross-tool-skills], so no per-tool translation is required.

### 1. discover

Inventory what sources are available before extracting from any of them. Setup-time discovery is one-shot ("here's the ADRs folder, here's the transcripts directory, here's the PR template"). Recurring discovery is hook-driven: detect new sources or new entries in existing sources since the last run.

The output is a list of source locations the rest of the pipeline can target. For most teams this list is short and stable; the value of writing it down is making it explicit so an agent doesn't re-derive it on every run.

### 2. extract

Pull candidate findings from a source location. Use deterministic parsing where the source has structure (ADRs in a folder, commits, ADO work items via `az boards`, GitHub issues via `gh issue list`). Use AI semantic analysis where the source is prose (transcripts, free-form PRs, PRD sections).

A candidate finding is small: one sentence stating the choice, fact, rule, or deviation, plus a pointer back to the source. It is not a draft record yet. Keeping extract narrow makes the draft phase tractable.

### 3. draft

Shape candidates into the four-type model. Each draft includes:

- Filled frontmatter following the [base record schema](../model/01-base-memory-record.md) and the type-specific extension ([Decision](../model/02-decision.md), [PolicyRule](../model/03-policy-rule.md), [Exception](../model/04-exception.md), [Context](../model/05-context.md))
- A short prose body following the type's body conventions
- `source_refs` pointing back to the originating source

Draft is where AI does the most work and where errors are most consequential, since downstream phases inherit whatever the draft asserts. The mitigations: keep candidates narrow, prefer extractive phrasing over inferred phrasing, and surface low-confidence drafts explicitly so the reconcile phase can flag them for closer review.

### 4. reconcile

Compare drafts against the existing bank state. For each draft, annotate one of:

- **Net new.** No matching record found. Forward to propose as-is.
- **Duplicate of `<existing-id>`.** The same content already exists; suggest dropping the draft or merging notable details.
- **Supersedes `<existing-id>`.** The draft conflicts with an existing accepted record; mark the existing one as a supersession candidate and link them.
- **Vocabulary mismatch.** The draft uses different scope terms than the existing bank (e.g., `enterprise` vs `ent`); flag for terminology resolution before propose.

Reconcile is structurally a code review against existing records. Treat the existing bank as the authoritative source; drafts argue their way in.

### 5. propose

Surface reconciled drafts for human review. The standard mechanism is a PR against the memory bank repo with `status: proposed` set in each draft's frontmatter. Each draft includes:

- The full record file
- Source provenance (link to the originating source)
- Reconcile annotations (net-new, supersedes, vocabulary notes)
- Suggested reviewers based on the `owners` field

Auto-merge is not allowed at this phase. Status moves to `accepted` only after a human reviewer approves and merges.

### 6. verify

After a proposed record is accepted, confirm Copilot can find it. The cheapest verification is round-tripping the source question: ask the question the record was meant to answer; the record should appear in citations. See [Leading Practices: Verifying Your Records](leading-practices.md#verifying-your-records) for the procedure across IDE, CLI, and Spaces surfaces.

If the record doesn't surface, treat it as a frontmatter problem first (most often `applies_to` vocabulary mismatch, less often `status`) before assuming the record itself is wrong.

## Sources, by Type

Different sources have different extraction profiles. The table is a starting menu; a team's `discover` output names specific instances.

| Source type | Best extracted by | Most often produces |
| --- | --- | --- |
| ADRs folder | Deterministic parse of frontmatter + AI summarization of body | Decision; occasionally PolicyRule when an ADR codifies a standing pattern |
| PR descriptions | AI summarization of prose, deterministic parse of linked issues/tickets | Decision (implementation choice with reach), Exception (sanctioned deviation) |
| Commit messages | Deterministic regex on conventional-commits prefixes | Context (gotchas, vendor behavior); occasionally Decision |
| Meeting transcripts | AI semantic extraction with low confidence by default | Decision (under discussion → propose for review), Context (facts established mid-conversation) |
| PRDs / scope docs | AI semantic extraction | Context (customer commitments, scope boundaries), Decision (prioritization, scope) |
| Issue trackers (`gh issue`, `az boards`) | Deterministic CLI fetch + AI summarization | Decision, Context; occasionally Exception |
| Existing wiki / docs | AI extraction; treat output as low confidence | Context most commonly; occasionally PolicyRule |

The fixture set in [`examples/hydration-demo/`](../examples/hydration-demo/) walks through several of these against a generic project. Use it as a worked example or as a substrate for a coaching session.

## The Bridge from Working Memory

A separate source type deserves its own section: the working-memory layer.

A `working-memory-kit` installation maintains a six-file working memory in each project: active context, project overview, decision log, data contracts, conventions, open questions. Some of those notes are durable enough to graduate into the memory bank as Decision, PolicyRule, Exception, or Context records.

Promotion reuses phases 2–5 of this pipeline with three differences:

- **Source.** `extract` reads working memory files (`decisionLog.md`, `conventions.md`, `dataContracts.md`, `openQuestions.md`). `activeContext.md` is local-only and not a promotion source.
- **Signal.** Promotion threshold is recurrence and broader applicability, not first-mention. A working memory note solved more than once across projects is a stronger candidate than a first-occurrence note.
- **Risk.** Reconcile has higher stakes. A working memory note that promotes without abstraction stays project-specific in the bank, defeating the cross-team value.

For the producing side (surfacing promotion candidates from working memory), see the parallel hydration page in `working-memory-kit`. The principle: promotion has friction by design. The human gate at propose is what keeps the bank's signal-to-noise high.

## What This Page Does Not Do

- It does not replace the [per-persona hydration guides](by-persona/). Those cover authorship triggers in role voice, field-fill cheat sheets, and worked records. AI-assisted hydration is a complementary path, not a replacement.
- It does not auto-merge. Every phase ends with a human-reviewable artifact; no record reaches `accepted` without human approval.
- It does not specify the executable details of each skill. Those live in [`.claude/skills/`](../.claude/skills/), one `SKILL.md` per phase.

## See Also

- [Retrieval](retrieval.md) for how Copilot finds records
- [Leading Practices](leading-practices.md) for frontmatter discipline, lifecycle, and verification
- [Per-persona hydration guides](by-persona/) for role-specific authorship
- [`SCAFFOLD.md`](../SCAFFOLD.md) for setting up a memory bank from scratch

[^cross-tool-skills]: Agent skills use a shared `SKILL.md` open format (`name` and `description` frontmatter plus a body). VS Code Copilot reads project skills from `.github/skills/`, `.claude/skills/`, and `.agents/skills/`. Claude Code reads them from `.claude/skills/`. Hosting at `.claude/skills/` covers both tools without duplication. See [VS Code: Use Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills); [GitHub Docs: Adding agent skills for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills).
