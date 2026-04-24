# Memory Bank Instructions

This repository uses a memory bank: a governed knowledge layer of decisions, rules, exceptions, and environmental facts. When working in this repo, consult the memory bank before inferring intent from code.

## Repository Structure

The memory bank lives under `memory-bank/` with records organized by type:

- `memory-bank/decisions/`: Decision records. Choices made, with alternatives considered.
- `memory-bank/rules/`: PolicyRule records. Standing guidance that should govern generated code.
- `memory-bank/exceptions/`: Exception records. Sanctioned deviations from PolicyRules.
- `memory-bank/context/`: Context records. Environmental facts others depend on.

Each record is a Markdown file with YAML frontmatter at the top. The frontmatter is authoritative for retrieval; the body is human context.

## Finding Relevant Records

Before generating or editing code, query the memory bank. Follow the four-stage retrieval funnel, cheapest first:

1. **Filter by type and scope.** Use `Glob` or directory listing to narrow by record type and filename slug before loading any content.
2. **Filter on frontmatter.** Use `Grep` against frontmatter fields (`applies_to`, `status`, `tags`) to narrow the candidate set without loading bodies.
3. **Read candidate headers.** Load just the YAML frontmatter of survivors (roughly the first 15 lines per file).
4. **Read full bodies.** Only for the three to five records that actually matter for the task at hand.

Do not read every record in full to find matches. If you catch yourself doing that, go back to stage 1.

## What Each Record Type Means for Your Behavior

- **Decision.** A retrospective choice. Explains why things are the way they are. When generating code inside a Decision's `applies_to` scope, respect the chosen approach unless the user explicitly overrides.
- **PolicyRule.** Prospective guidance. The `enforcement` field determines how strictly to treat it:
  - `required`: hard constraint. Do not generate code that violates it without an Exception record covering the scope.
  - `recommended`: expected default. Flag or document deviations.
  - `advisory`: surface as context; do not block on it.
- **Exception.** A sanctioned carve-out from a PolicyRule. If an Exception's `applies_to` matches your scope and its `exception_to` points at a PolicyRule in scope, the Exception governs inside its `scope_boundary`.
- **Context.** An environmental fact. Reason over it; do not contradict it. The `constraints` field describes implications for downstream work.

## Status and Supersession

Records carry a lifecycle through `status`: `proposed → accepted → [superseded | deprecated | rejected]`.

- `proposed`: under discussion. Surface as context; do not treat as binding.
- `accepted`: authoritative. Act on it.
- `superseded`: replaced. Follow `superseded_by` to the replacement UUID before acting. Do not act on the stale record.
- `deprecated`: no longer applies. Do not act on it.
- `rejected`: considered and declined. Do not act on it.

Supersession is a two-record operation: the old record gets `status: superseded` and `superseded_by: <uuid>`; the new record gets `supersedes: [<uuid>]`.

## What Not to Do

- Do not invent memory bank records. Records are authored by the team; your job is to surface and reason over them.
- Do not update a record in place to reflect a new decision. Write a new record and mark the old one `superseded`.
- Do not treat `proposed` records as binding constraints.
- Do not bundle multiple decisions into one record, and do not author records that restate what the code already says.

## When the User Is Authoring a Record

If the user is writing a memory bank record, point them at the role-specific guide:

- Architects: `memory-bank/guide/by-persona/architects.md`
- PMs / Product Owners: `memory-bank/guide/by-persona/pms.md`
- Developers: `memory-bank/guide/by-persona/developers.md`

For cross-cutting habits (frontmatter discipline, lifecycle, verification), see `memory-bank/guide/leading-practices.md`.

If the memory bank includes templates at `memory-bank/_templates/{type}.md`, start the new record from the matching template; templates encode required fields and structure.

---

## Customizing This File

This is a generic starter. Before deploying it as `.github/copilot-instructions.md` in your repo, adjust:

- **Directory paths.** The paths above assume the default scaffold layout (`memory-bank/decisions/`, `memory-bank/rules/`, and so on). Change them if your team uses per-role or per-domain organization.
- **Exception tracking.** If your team does not track Exceptions as a separate type, remove Exception references.
- **Scope vocabulary.** If your team scopes records with specific vocabulary (service names, domain names, customer segments, system IDs), add a short section naming that vocabulary so the agent can match queries to records.
- **Role coverage.** Remove persona-guide links for roles your team does not have.
- **Extensions.** Add team-specific guidance as needed: preferred tools, naming conventions, additional anti-patterns to avoid.
