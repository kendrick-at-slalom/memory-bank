---
description: Discipline for content in the examples tree.
applyTo: 'examples/**'
---

# Working in examples/

This is a reference architecture, not an engagement artifact. Examples must:

- **Use generic names.** No client names, no engagement-specific content, no proprietary domain language. If you need a service name, prefer something like `order-service` over a real one.
- **Be illustrative, not exhaustive.** Pick the minimum example that demonstrates the concept; do not pad with realistic-looking-but-fake content. Length is not credibility.
- **Follow the same frontmatter discipline as production records.** Examples are reference reading material for new authors; sloppy frontmatter here teaches bad habits.
- **Cross-reference with `examples/README.md`.** Records that demonstrate relationships (Decision citing PolicyRule via `constrained_by`, Exception linked to PolicyRule via `exception_to`, supersession chains) should be called out in that index so readers can find them.

When adding a new example, ask whether it teaches something the existing examples don't. If it duplicates an existing demonstration, edit the existing one instead.

The Claude-format equivalent of this file is [`examples/CLAUDE.md`](../../examples/CLAUDE.md). Keep both in sync when changing this guidance.
