---
status: v0 draft
last_updated: 2026-04-23
---

# Memory Bank Hydration Guide

This guide walks you through setting up and populating a memory bank so that your AI coding assistant can find and reason over your team's decisions, rules, exceptions, and environmental facts at generation time.

The model itself (record types, field definitions, relationships) lives in [`model/`](../model/). This guide covers the practical side: what to do first, how to write records in your role, and how to verify that Copilot can actually find them.

## Who This Is for

Anyone hydrating a memory bank for the first time. The guide assumes you have a Copilot license (any tier) and a Git repo where the memory bank will live.

Role-specific sections cover architects, product managers, and developers. The four record types are the same everywhere, but the moments that produce records and the language that fills the fields look different depending on your role.

## Where to Start

If you're new to this:

1. **This page** for what a memory bank is, the model at a glance, and how to write your first record.
2. **[Retrieval](retrieval.md)** for how Copilot finds records and the retrieval funnel.
3. **Your role's hydration guide** for when to write records, how to fill the fields in your voice, and worked examples:
   - [Architects](by-persona/architects.md)
   - [Product managers / product owners](by-persona/pms.md)
   - [Developers](by-persona/developers.md)
4. **[Leading Practices](leading-practices.md)** for cross-cutting discipline: frontmatter habits, lifecycle, cross-role work, anti-patterns, verification.

If you want the full schema details, see [`model/README.md`](../model/README.md).

---

## What Is a Memory Bank?

A memory bank is a governed knowledge layer that your AI coding assistant consults at generation time. It's a collection of Markdown files with YAML frontmatter, organized so that agents can find the right records cheaply without reading everything.

What it is not:

- Not an agent-memory infrastructure product (Mem0, Zep, etc.). There's no database, no embedding store, no running service.
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

## Authorship Map

The four types apply to every role. What changes is the moment that produces the record and the language that fills the fields.

| Persona                | Decision                                                                            | Context                                                                          | PolicyRule                                                              | Exception                                                           |
| ---------------------- | ----------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Architect**          | Architectural choice made under constraints (event sourcing, persistence, topology) | System fact learned or confirmed (air-gapped network, canonical source of truth) | Standing architectural pattern (Postgres standard, mTLS baseline)       | Granting or documenting a sanctioned technical deviation            |
| **PM / Product Owner** | Prioritization call, scope boundary, launch sequencing                              | Customer/market fact, segment truth, commitment (tag as `commitment`)            | Product principle (self-serve promise, tier gating)                     | Customer- or release-specific carve-out against a product principle |
| **Developer**          | Library/tool choice, implementation pattern with reach                              | Gotcha, performance characteristic, vendor behavior                              | Team practice codified as a standard (all HTTP calls through a wrapper) | Workaround accepted against a team or enterprise standard           |

The deeper version (triggers, field-fill cheat sheets, and full worked records for each cell) lives in the per-persona hydration guides: [Architects](by-persona/architects.md), [PMs / POs](by-persona/pms.md), [Developers](by-persona/developers.md).

## Frontmatter Is the Interface

Structured frontmatter determines whether Copilot can find your record. The body is for humans; the frontmatter is for the agent. See [Leading Practices: Frontmatter Discipline](leading-practices.md#frontmatter-discipline) for the full set of habits.

## Steering Copilot with `copilot-instructions.md`

A `.github/copilot-instructions.md` file[^copilot-instructions] in your repo tells Copilot how to work with your memory bank. IDE Copilot, Copilot CLI, and the Copilot coding agent all read it automatically. You configure once; all surfaces pick it up.

A starter template is available at [`copilot-instructions.starter.md`](../copilot-instructions.starter.md) in the repo root. The starter contains `<!-- SCAFFOLD:... -->` markers flagging spots that vary by team; run [`SCAFFOLD.md`](../SCAFFOLD.md) to customize and deploy the result to `.github/copilot-instructions.md` in your code repo, or consult SCAFFOLD.md's § "Agent Instructions File" for the replacement guide to do it manually.

If you use Copilot Spaces[^copilot-spaces], the Space scoping works alongside `copilot-instructions.md` — the instructions tell Copilot _how_ to use the memory bank, the Space tells it _which repos to look in_.

## Physical Organization

Before you write records, pick how your memory bank is organized. The structure determines where files live, which is the first-level retrieval filter.

| Option               | Structure                                                 | Best for                                                |
| -------------------- | --------------------------------------------------------- | ------------------------------------------------------- |
| A: Per-role/function | `architecture-memory-bank/`, `product-memory-bank/`, etc. | Federated teams, distinct governance per role           |
| B: Per-domain        | `commerce-memory-bank/`, `platform-memory-bank/`, etc.    | Strong domain ownership, cross-functional collaboration |
| C: Single repo       | `memory-bank/decisions/`, `memory-bank/context/`, etc.    | Small teams, single products, individual use            |

The [scaffold prompt](../SCAFFOLD.md) helps you choose based on your situation.

## Writing Your First Record

Start with a Context record. It's the simplest type: a fact about your environment that other roles need at generation time.

1. Create a file in the appropriate directory (e.g., `context/platform-CTX-0001.md`)
2. Fill the frontmatter. At minimum: `id`, `title`, `memory_type: Context`, `status: active`, `applies_to`, and the Context-specific `fact_statement` field.
3. Write a brief body explaining the fact in prose. The body supports the frontmatter; it doesn't replace it.
4. Commit and push.
5. Verify Copilot can find it; see [Leading Practices: Verifying Your Records](leading-practices.md#verifying-your-records).

For a complete worked example of a Context record, see [`examples/context/`](../examples/context/).

Once you've written your first record, read the [retrieval guide](retrieval.md) to understand how Copilot finds records. Then jump to your role's hydration guide for triggers and worked examples, and [leading practices](leading-practices.md) for verification and cross-cutting discipline.

[^copilot-instructions]: `.github/copilot-instructions.md` is GitHub Copilot's repository-scoped custom instructions file. Instructions in the file are automatically added to Copilot requests whenever the repo is in scope — across the IDE extensions, the coding agent on github.com, and the Copilot CLI. See [GitHub Docs: Adding repository custom instructions for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/configure-custom-instructions/add-repository-instructions); [GitHub Docs: Adding custom instructions for GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions).

[^copilot-spaces]: Copilot Spaces let you organize the context Copilot uses to answer questions — repositories, pull requests, issues, free-text notes, and file uploads — and share that scoped context with a team. Spaces are available to any Copilot license tier, including Copilot Free. See [GitHub Docs: About GitHub Copilot Spaces](https://docs.github.com/en/copilot/concepts/context/spaces).
