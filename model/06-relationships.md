# Relationships

## Why Relationships Matter

A memory bank of isolated records is a filing cabinet. A memory bank where records link to each other is a graph — and a graph is what lets an agent answer questions that span multiple records.

Consider: _"Why are we using Redis for the session cache when our persistence standard is Postgres?"_ Answering requires pulling together a PolicyRule (Postgres is standard), an Exception (session cache is exempted), a Context (why session cache has different requirements), and a Decision (why Redis specifically). Without explicit relationships, the agent infers connections from prose. With them, it follows links.

Relationships also matter for _retrieval scope_. When an agent pulls a record, it should usually pull immediately linked records too. A Decision with linked Contexts, PolicyRules, and alternatives is embedded in its proper context.

The trick is keeping the vocabulary small enough to be usable and large enough to express the patterns that actually matter.

---

## The Relationship Vocabulary

Six types:

| Relationship | What it means | Example |
|---|---|---|
| `supersedes` / `superseded_by` | Lifecycle replacement | A new ADR supersedes an old one |
| `constrained_by` | Bound by another record (usually PolicyRule or Context) | A Decision is constrained by a PolicyRule mandating Postgres |
| `derived_from` | Logical consequence or codification | A PolicyRule derived from an earlier Decision |
| `depends_on` | Assumes another record is true | A Decision depends on a Context about Kafka availability |
| `informs` | Shaped by but not bound | A Decision informed by user-research Context |
| `relates_to` | Loose connection, fallback | Two adjacent Decisions that aren't in a specific logical relationship |

That's the full set. Any relationship between records should fit one of these six. If it doesn't, `relates_to` is the honest answer.

---

## How Relationships Are Expressed

### Top-level Lifecycle Fields

The base schema has dedicated fields for supersession because it's common and important:

```yaml
supersedes: [<uuid>]
superseded_by: null
```

### The `related` List

Everything else goes in `related`:

```yaml
related:
  - uuid: 8a2f1e4c-7b9d-4c3a-8e5f-1d2a3b4c5d6e
    relationship: constrained_by
  - uuid: 2b4f8a1c-6e3d-4f7a-9b2c-5d8e1a4f7b3c
    relationship: depends_on
```

Agents always link by UUID. The human-facing `id` is for conversation; the UUID is stable across renames and federation.

---

## Bidirectionality

Relationships have direction but are meaningful in both. If A is `constrained_by` B, then B constrains A.

**Store forward links explicitly; derive backward links at query time.**

- A stores `{uuid: B, relationship: constrained_by}`.
- B does not store a corresponding link.
- Querying B for "what does this constrain?" searches for records pointing at B.

Advantages:
1. No risk of divergence (single source of truth).
2. Less writing (only update one record).
3. Backward queries stay cheap (frontmatter is already being scanned).

**Exception: supersession.** Both sides store the link explicitly because the relationship changes lifecycle state. The superseded record's `status` changes, so it must know it's been superseded without requiring a backward search.

---

## Supersession in Detail

### What It Is

Replacing an older record with a newer one. The older record is preserved for history but stops being authoritative. An agent answering "what's current?" follows the chain forward.

Supersession is always deliberate — a record doesn't become superseded just because a newer one exists that says something different.

### The Operation

When record B supersedes record A:

1. **Record A updates.** `status` → `superseded`. `superseded_by` → UUID of B. `effective_to` → date of supersession.
2. **Record B is written.** `supersedes` → `[uuid_of_A]`. `effective_from` → same date.
3. **Both records remain.** A is not deleted.

Any step missing creates a broken supersession:
- B supersedes A but A isn't updated → A still looks current
- A is updated but B doesn't link back → orphaned link
- A is deleted → history lost, auditability gone

**Supersession is always a two-record operation.**

### Partial Supersession

A new record supersedes an old one for a specific scope only. The old record stays `status: accepted` with its broader scope. The new record declares `supersedes` but constrains its own `applies_to`. Agents use the more-specific record when the scope matches.

### Supersession Chains

Records can form chains: A → B → C (current). Agents follow forward until they hit a non-superseded record. Each superseded record points at its immediate successor, not the final version.

Long chains (>3-4) are a smell — either consolidate or check whether the underlying question needs reframing.

---

## Common Patterns

### Decision Derived from Context

A Decision shaped by an environmental fact:

```yaml
# In the Decision
related:
  - uuid: <context-uuid>
    relationship: informs
```

Use `informs` when Context shaped thinking but didn't force outcome. Use `constrained_by` when Context made the outcome unavoidable.

### Decision Constrained by PolicyRule

A Decision made under a standing rule:

```yaml
# In the Decision
related:
  - uuid: <policy-rule-uuid>
    relationship: constrained_by
```

If the Decision appears to violate the PolicyRule, check for an Exception. No Exception = non-compliance.

### PolicyRule Derived from Decision

A rule codified from an earlier choice:

```yaml
# In the PolicyRule
related:
  - uuid: <decision-uuid>
    relationship: derived_from
```

### Exception Points at PolicyRule

The defining relationship for Exceptions:

```yaml
# In the Exception
exception_to: <policy-rule-uuid>
```

This is a required field, not a `related` entry.

### Context Depends on Context

Contexts often rest on other Contexts:

```yaml
# In the dependent Context
related:
  - uuid: <foundational-context-uuid>
    relationship: depends_on
```

### Cross-role Relationships

Records from different roles can and should link. A product Context can inform an architecture Decision. A support PolicyRule can constrain a developer Decision. Cross-role links are where the memory bank's value compounds.

---

## Records Without Links

A record with zero relationships isn't necessarily wrong, but pause on it. Most records in a mature memory bank link to at least one other record.

Orphan records tend to be:
- **Early records** in a new memory bank (nothing to link to yet)
- **Foundational records** at the root of the knowledge graph
- **Low-value records** that probably shouldn't exist

When writing a record, try to name at least one relationship. If you can't, either the memory bank is too sparse or the record isn't useful.

---

## Retrieval Behavior

When an agent retrieves a record, it should also retrieve one level of linked records (outbound links in `related`, `supersedes`, `superseded_by`, `exception_to`). This produces coherent context.

Don't follow links from linked records — that way lies context explosion. One level is enough for most queries. Follow chains further only when the query explicitly asks for history or lineage.

---

## Common Mistakes

**Using `relates_to` for everything.** It carries no semantic weight. Prefer a more specific type when one fits.

**Forgetting to link supersession on both sides.** Both records must be updated.

**Linking to records that don't exist.** Validate at write time.

**Deep chains when a rewrite would do.** A Decision superseded four times might benefit from a single consolidating record.

**Treating relationships as documentation.** Links in prose ("see ADR-0042") are invisible to agents. Declare in frontmatter.

**Inferring instead of declaring.** A Decision that implicitly depends on a Context isn't linked unless the author declares it.

**Over-linking.** A record with twenty links is usually a record with three meaningful links and seventeen decorative ones. Link what matters for the record's meaning.
