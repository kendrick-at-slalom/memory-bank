---
status: v0 draft
last_updated: 2026-04-23
---

# Memory Bank Hydration Guide

This guide walks you through setting up and populating a memory bank so that your AI coding assistant can find and reason over your team’s decisions, rules, exceptions, and environmental facts at generation time.

The model itself (record types, field definitions, relationships) lives in [`model/`](../model/). This guide covers the practical side: what to do first, how to write records in your role, and how to verify that Copilot can actually find them.

## Who This Is for

Anyone hydrating a memory bank for the first time. The guide assumes you have a Copilot license (any tier) and a Git repo where the memory bank will live.

Role-specific sections cover architects, product managers, and developers. The four record types are the same everywhere, but the moments that produce records and the language that fills the fields look different depending on your role.

## Where to Start

If you’re new to this:

1. **This page** — what a memory bank is, the model at a glance, and how to write your first record
2. **[Retrieval](retrieval.md)** — how Copilot finds records, the retrieval funnel, and how to verify your records are findable
3. **Your role’s hydration guide** — when to write records, how to fill the fields in your voice, and worked examples:
   - [Architects](by-persona/architects.md)
   - [Product managers / product owners](by-persona/pms.md)
   - [Developers](by-persona/developers.md)

If you want the full schema details, see [`model/README.md`](../model/README.md).

---

## What Is a Memory Bank?

A memory bank is a governed knowledge layer that your AI coding assistant consults at generation time. It’s a collection of Markdown files with YAML frontmatter, organized so that agents can find the right records cheaply without reading everything.

What it is not:

- Not an agent-memory infrastructure product (Mem0, Zep, etc.). There’s no database, no embedding store, no running service.
- Not a wiki. Records have structured frontmatter that agents filter on; wikis are prose that agents have to read in full.
- Not a documentation repo. Docs describe how things work. Memory bank records capture why things are the way they are, what rules apply, and where exceptions exist.

## The Four Record Types

Every record in the memory bank is one of four types. They share a common base schema ([`model/01-base-memory-record.md`](../model/01-base-memory-record.md)) and add type-specific fields.

| Type           | What it captures                                                         | When you write one                                                                                                               |
| -------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| **Decision**   | A choice made, with alternatives considered and constraints acknowledged | After a design review, a prioritization call, a library choice — any moment where you picked one option over others              |
| **PolicyRule** | A standing rule that applies until explicitly changed                    | When a pattern is codified, a standard is adopted, or a governance constraint is established                                     |
| **Exception**  | A sanctioned deviation from a PolicyRule                                 | When a team gets permission to do something differently, with a review date                                                      |
| **Context**    | An environmental fact that other records depend on                       | When you learn something about a system, a customer, a vendor, or a constraint that other people need to know at generation time |

For full type definitions: [`model/02-decision.md`](../model/02-decision.md), [`model/03-policy-rule.md`](../model/03-policy-rule.md), [`model/04-exception.md`](../model/04-exception.md), [`model/05-context.md`](../model/05-context.md).

## Frontmatter Is the Interface

Structured frontmatter fields determine whether Copilot can find your record. A record with thin frontmatter is invisible to the scope pass regardless of how well the body is written. The body is for humans; the frontmatter is for the agent.

The most important field is `applies_to`. If you fill nothing else beyond the required fields, fill that one.

## Steering Copilot with `copilot-instructions.md`

A `.github/copilot-instructions.md` file in your repo tells Copilot how to work with your memory bank. IDE Copilot, Copilot CLI, and the Copilot coding agent all read it automatically. You configure once; all surfaces pick it up.

A starter template is available at [`copilot-instructions.md`](../copilot-instructions.md) in the repo root. Copy it into your memory bank repo’s `.github/` directory and adjust the values for your team.

If you use Copilot Spaces, the Space scoping works alongside `copilot-instructions.md` — the instructions tell Copilot _how_ to use the memory bank, the Space tells it _which repos to look in_.

## Writing Your First Record

Start with a Context record. It’s the simplest type: a fact about your environment that other roles need at generation time.

1. Create a file in the appropriate directory (e.g., `context/platform-CTX-0001.md`)
2. Fill the frontmatter. At minimum: `id`, `title`, `memory_type: Context`, `status: active`, `applies_to`, and the Context-specific `fact_statement` field.
3. Write a brief body explaining the fact in prose. The body supports the frontmatter; it doesn’t replace it.
4. Commit and push.
5. Verify Copilot can find it — see [Retrieval: verifying a record is findable](retrieval.md#verifying-a-record-is-findable).

For a complete worked example of a Context record, see [`examples/context/`](../examples/context/).

Once you’ve written your first record, read the [retrieval guide](retrieval.md) to understand how Copilot finds records and how to verify yours are working. Then jump to your role’s hydration guide for triggers and worked examples.
