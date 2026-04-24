---
status: v0 draft
last_updated: 2026-04-23
---

# Leading Practices

Cross-cutting practices that apply to any team running a memory bank, regardless of role. These are the discipline pieces: frontmatter habits, lifecycle, cross-role work, anti-patterns, and verification.

For role-specific tips, see your persona page:

- [Architects](by-persona/architects.md#tips)
- [PMs / POs](by-persona/pms.md#tips)
- [Developers](by-persona/developers.md#tips)

## Frontmatter Discipline

Structured frontmatter fields determine whether Copilot can find your record. A record with thin frontmatter is invisible to the scope pass regardless of how well the body is written. The body is for humans; the frontmatter is for the agent.

Three habits, in order of impact:

- **Always fill `applies_to`.** It's the highest-value retrieval filter. If your record doesn't name the services, domains, segments, or systems it governs, queries scoped to those things will miss it. If you fill nothing else beyond the required fields, fill this one.
- **Use consistent vocabulary.** Agree on scope names and tags across your team. Calling a segment "enterprise" in one record and "ent" in another means queries using one term will miss records using the other. This matters even more when the bank serves multiple personas, since cross-role retrieval depends on the words lining up.
- **Set `status` deliberately.** Records sit in `proposed` until the team accepts them; once accepted, they're authoritative. Don't leave everything in `draft` or `proposed` indefinitely. An agent reading a `draft` record treats it as not-yet-binding, which is probably not what you want.

## Record Lifecycle

Records are governed, not static. The lifecycle:

```
proposed → accepted → [superseded | deprecated | rejected]
```

Three habits that keep the bank honest over time:

- **Supersede, don't edit in place.** When a decision changes, write a new record and mark the old one `superseded` with `superseded_by` pointing to the replacement. Don't rewrite the old record; its body is valuable history. Supersession is a two-record operation: the old record gets `superseded` + `superseded_by`, the new record gets `supersedes`.
- **Put review dates on Exceptions.** `review_by` is how you prevent carve-outs from quietly becoming permanent. An Exception without a review date becomes a shadow rule that nobody revisits.
- **Superseded and deprecated records stay in the bank.** They're not deleted. They answer "why did we change?" queries and preserve the reasoning chain. Agents resolve the supersession chain to find the current version.

For the full lifecycle rules and per-type specifics, see [`model/01-base-memory-record.md`](../model/01-base-memory-record.md).

## Working Across Roles

The memory bank gets its cross-role leverage from explicit links between records authored by different personas.

- **Use relationships, don't hope.** A PM's Context about a customer commitment can show up in an architect's Decision via `constrained_by`; a developer's Decision can cite an architect's PolicyRule the same way. The [relationships vocabulary](../model/06-relationships.md) gives you the link types; use them explicitly rather than leaving readers to connect the dots.
- **Agree on scope vocabulary across personas.** Retrieval works across persona boundaries only when the words line up. If PMs call a segment "enterprise" and architects call it "ent," the architect's query for records about "ent" won't find the PM's Context. Pick the terms once as a team and stick to them.
- **One record per decision, fact, or rule.** Don't bundle several decisions into one record because they came from the same meeting. Agents retrieve at the record level; bundled records dilute retrieval and make supersession messy.
- **Decisions that cite PolicyRules via `constrained_by` are more useful than standalone Decisions.** The link tells Copilot which rules shaped the choice and lets future readers walk from choice to constraint.
- **When you write a PolicyRule, plan for future Exceptions.** If anyone might need an Exception against the rule someday (the answer is usually yes), make sure the PolicyRule's `id` is memorable and stable. Exceptions with unclear `exception_to` targets are hard for agents to link back.

## What Not to Build

Three common anti-patterns that look helpful and aren't:

- **Don't maintain a manual index file.** The filesystem (or Copilot) answers "what's in the bank?" on demand. An index drifts every time a record is added, renamed, or superseded, and it teaches readers to consult a stale document instead of the real thing. Generate inventories when you need them: `grep "^title:" decisions/` gives a list of decision titles; the Copilot equivalent is asking it to summarize what's in the repo.
- **Don't duplicate records across persona repos.** If two personas reference the same fact, each links to one record rather than authoring their own copy. Duplicates drift independently.
- **Don't use the memory bank as a task tracker or meeting notes archive.** Records capture _decisions, rules, exceptions, and facts_. Task trackers capture _work_. Meeting notes capture _conversation_. Mixing them weakens retrieval and pollutes the corpus.

## Verifying Your Records

After writing a record, confirm it's retrievable before calling it hydrated. Ask Copilot the question the record was meant to answer. The record should appear in the citations.

You can verify from any surface:

- **Copilot Spaces** (for GitHub-hosted memory bank repos): ask in your Space.
- **IDE Copilot** (all teams): open the memory bank repo, ask the question. `copilot-instructions.md` guides Copilot to the right files.
- **Copilot CLI** (all teams): run from the repo directory. Same `copilot-instructions.md`, full file access.

If the record doesn't surface:

- **Check `applies_to`.** Empty `applies_to`, or values that use different vocabulary than your question, make the record invisible to the scope pass.
- **Check `status`.** Records in `proposed` state may be filtered out depending on how the agent is configured.
- **Check that the record is in the right Space** (if using Spaces) or the right directory.
- **Check vocabulary mismatch.** If the question uses "enterprise" and the record tags say "ent," they won't connect.

Fix the frontmatter and re-ask. If the record surfaces, it's hydrated correctly. If it still doesn't, the question you're asking probably doesn't line up with the vocabulary you tagged; adjust the tags, or adjust how your team phrases the question, and try again.

For the full retrieval model and surface-by-surface mapping, see [Retrieval](retrieval.md).
