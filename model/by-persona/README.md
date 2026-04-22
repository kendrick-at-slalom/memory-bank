# Memory Bank Authorship, by Persona

The main schema files (`01`–`06` at the parent level) define what each record type is and how its fields work. This subdirectory answers a different question: **for you, in your role, when do you author a record and what does it look like?**

The schema is role-agnostic by design; the moments that produce records and the language that fills the fields are different for each role. These guides give you the view from inside your own work.

## Files

- [`architect-authorship.md`](architect-authorship.md) — authorship triggers and worked examples for architects
- [`pm-authorship.md`](pm-authorship.md) — authorship triggers and worked examples for product managers and product owners
- [`developer-authorship.md`](developer-authorship.md) — authorship triggers and worked examples for developers

Each guide follows the same shape: authorship triggers for all four record types, a field-fill cheat sheet showing how you naturally phrase each field, and one full worked record per type in your voice.

## How to use these guides

**If you're starting out**, read your persona's guide before reading the schema files. The worked examples in your voice make the schema files land when you get to them.

**If you're trying to figure out whether something you just did becomes a record**, jump to the authorship triggers section in your persona's guide. The table shows what moments produce what types.

**If you know you want to write a Decision (or Context, PolicyRule, Exception) but aren't sure how the fields apply to your work**, the field-fill cheat sheet in your guide gives you the shorthand. The schema file gives you the full definition; your persona guide gives you the voice.

## The model is shared; the voice is yours

The four record types (Decision, Context, PolicyRule, Exception) apply to every persona. A PM's Decision has the same shape as an architect's Decision: `decision_question`, `decision_outcome`, `alternatives_considered`, and so on. What changes is the content. These guides exist so you don't have to translate architecture-flavored examples into your own work every time you read the schema.
