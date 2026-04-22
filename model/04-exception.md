# Exception

## What an Exception Is

An `Exception` record captures a sanctioned deviation from a PolicyRule: _we broke the rule, intentionally, here's why, and here's who approved it._

Exceptions are the shortest type in terms of content but arguably the highest-value type for trust and audit. They turn "somebody told me it was okay" into a first-class, auditable record. In environments with safety regimes, cybersecurity standards, long-lived systems, or formal change control, exceptions are not optional bookkeeping, they're how the organization proves it's managing risk consciously rather than accidentally.

Exceptions exist _only_ in reference to PolicyRules.[^togaf-governance] You cannot have an Exception without the PolicyRule it excepts. If there's no rule to point at, you have a Decision or a Context instead.

---

## Why You Should Track Exceptions

It's tempting to fold exceptions into Decisions or handle them informally. We strongly recommend against this:

- **Visibility.** An Exception is a first-class record type that agents can filter on (`memory_type: Exception`). A decision that happens to be an exception requires reading the body to know it's a deviation. That means _agents can't find it_ during the scope pass.
- **Auditability.** When an auditor asks "show me all deviations from standard X," well-structured Exception records answer in seconds. Informal records answer in weeks.
- **Time-bounding.** Exception records have `review_by` dates that force periodic renewal. Informal exceptions become permanent by default.
- **Scope control.** Explicit `scope_boundary` fields prevent exceptions from silently expanding to cover more than they were granted for.
- **Compensating controls.** The structure forces you to articulate what replaces the rule's protection, which forces honest thinking about whether the exception is safe.

Teams that skip exception tracking accumulate shadow rules that silently erode their standards.[^shadow-it] Every "We got permission" that lives only in Slack is an invisible hole in governance.

---

## How Exceptions Differ from the Other Types

- **Exception vs. PolicyRule.** A PolicyRule says "Do X." An Exception says "For this case, you don't have to." They're complementary.
- **Exception vs. Decision.** A Decision is a choice made freely. An Exception is a choice made _against_ a standing rule. The test: if there were no rule, would this record still exist? If yes, Decision. If it only exists because a rule had to be broken, Exception.
- **Exception vs. Context.** Context describes the environment. An Exception grants permission to deviate. "The network is air-gapped" is Context; "Services on the air-gapped network are exempted from mTLS" is an Exception that cites the Context.

**Rule of thumb:** an Exception always has a PolicyRule in its `exception_to` field. If you can't fill it in, you don't have an Exception.

---

## What an Exception Looks Like for You

Exceptions feel architect-coded at first glance (air-gapped networks, mTLS carve-outs), but every persona that authors PolicyRules also authors the Exceptions against them. Same schema, same care required.

| Field | Architect | PM / Product Owner | Developer |
| --- | --- | --- | --- |
| `exception_to` | UUID of the architectural PolicyRule | UUID of the product principle | UUID of the team practice or enterprise standard |
| `justification` | Technical constraint that blocks compliance + what replaces the protection | Prior commitment or strategic circumstance + what substitutes for the principle's protection | Framework limitation, legacy constraint, runtime environment + what mitigates the gap |
| `compensating_controls` | Network segmentation, physical access review, equivalent technical controls | Manual CSM onboarding, 30-day check-in, precedent-logging Context | Per-request timeouts, alerting on error rates, linked retirement Decision |
| `review_by` | Annual for structural; shorter for transitional | Tied to contract window or commitment date | Tied to the retirement date of the workaround |
| `scope_boundary` | "Applies only to plant-03, not other plants" | "Applies only to customer-X, not other tier-2 customers" | "Applies only to legacy-billing-adapter, not other legacy modules" |

The authorship trigger: you write an Exception when you (or an approver in your scope) sanction a deviation from a standing rule for a specific case. The rule still stands; this record is how you prove the deviation is governed, bounded, and time-limited.

For full worked examples and authorship triggers by persona:

- [`by-persona/architect-authorship.md`](by-persona/architect-authorship.md)
- [`by-persona/pm-authorship.md`](by-persona/pm-authorship.md)
- [`by-persona/developer-authorship.md`](by-persona/developer-authorship.md)

---

## Exception-Specific Frontmatter

```yaml
---
# --- Base fields (see 01-base-memory-record.md) ---------------------
id: plant-floor-EXC-0003
uuid: 3c9d2e8a-5b7f-4a1c-9e6d-8f2a4b5c7d9e
memory_type: Exception
title: Plant-floor telemetry services exempted from mTLS requirement
status: accepted
owners:
  - role: ot-security-lead
    name: plant-floor-security
applies_to:
  services: [telemetry-collector, scada-bridge]
  systems: [plant-03-historian]
  domains: [ot-plant-floor]
tags: [mtls, security, air-gapped]
effective_from: 2026-03-15
effective_to: 2027-03-15
source_refs:
  - type: meeting
    ref: 'Security Review 2026-03-12'
  - type: ticket
    ref: 'SEC-0447'

# --- Exception-specific fields --------------------------------------

# Required for Exception
exception_to: 2b4f8a1c-6e3d-4f7a-9b2c-5d8e1a4f7b3c # uuid of the PolicyRule
justification: >
  Plant-03 historian and telemetry collectors run on an air-gapped network
  with no external connectivity. Certificate distribution requires
  connectivity to the enterprise PKI, which is unavailable by design.
  Network-layer controls and physical isolation provide equivalent protection.
approved_by:
  - role: ot-security-lead
    name: plant-floor-security
  - role: platform-arb
    name: platform-architecture-review-board

# Recommended for Exception
# review_by: forces exceptions to be revisited rather than living forever
review_by: 2027-03-01

# compensating_controls: what replaces the rule's protection
compensating_controls:
  - 'Air-gapped network segmentation enforced at physical layer'
  - 'Quarterly physical access review'
  - 'Local authentication with hardware tokens'

# scope_boundary: explicit edges of the carve-out
scope_boundary: >
  Applies only to services on the plant-03 OT network. Does not extend to
  any service with a network path to the enterprise network. Does not apply
  to other plants (separate exceptions required).

# Optional for Exception
risk_assessment: mitigated # accepted | mitigated | residual | unknown
incident_history: [] # uuids of any incident reports
---
```

---

## Field Reference (Exception-specific)

### Required

**`exception_to`**: UUID of the PolicyRule this exception applies to.

- The defining field. Without it, the record is not an Exception.
- Must reference an actual PolicyRule. An Exception can only point at one rule. Need exceptions from multiple rules? File multiple Exception records.
- If the PolicyRule has `exceptions_allowed: false`, the Exception is invalid.

**`justification`**: Why the exception is warranted.

- Must be substantive. Not "because we had to" or "legacy system."
- Should answer: (1) what specifically prevents compliance, (2) why that constraint can't be removed, (3) what mitigates the risk.
- Good: _"Plant-03 runs on an air-gapped network. Certificate distribution requires enterprise PKI connectivity, unavailable by design. Physical isolation provides equivalent protection."_
- Bad: _"Doesn't work with our environment."_

**`approved_by`**: Who approved the exception.

- Must match the PolicyRule's `exception_authority`.
- Must be a real, resolvable approver. "The team" is not acceptable.

### Recommended

**`review_by`**: When the exception must be revisited.

- _What's gained:_ Exceptions without review dates become permanent. This is the single worst failure mode.[^iso27001] A "temporary" exception that nobody revisits accumulates silently over years.
- Short-term exceptions: review in weeks or months. Structural exceptions: annual or biennial review to confirm conditions haven't changed.
- An exception without a review date is not a real exception — it's a hole in your standards.

**`compensating_controls`**: What replaces the rule's protection.

- _What's gained:_ A rule protects something. Removing that protection without replacement is accepting risk. This field forces honest thinking about safety.
- Format: flat list of short statements describing each control.

**`scope_boundary`**: Natural-language description of what's in and out.

- _What's gained:_ Exceptions leak. Scope creep is the second-worst failure mode after "never reviewed." Making boundaries explicit (especially "does NOT apply to...") reduces ambiguity.

### Optional

**`risk_assessment`**: Qualitative risk posture: `accepted`, `mitigated`, `residual`, `unknown`.

**`incident_history`**: UUIDs of incident reports tied to this exception.

---

## The Prose Body

```markdown
## Rule being excepted

Brief summary of the PolicyRule. Cite its id and title. Don't re-explain
in full — link to it.

## Circumstances

What specifically prevents compliance in this scope? Expanded justification.

## Justification

Why the deviation is warranted, including risk reasoning.

## Compensating controls

Detail each control: who operates it, how it's verified, how often checked.

## Scope

What's in, what's out, where the boundaries are. Include examples.

## Review and expiration

When it expires, what triggers review, what happens if conditions change.

## Approval history

Who approved, when, under what terms. Track renewals and modifications.
```

---

## How Agents Should Use Exceptions

**At generation time.** Before refusing on a PolicyRule violation, check whether an Exception in scope applies. If yes, proceed (mention the Exception). If no, the rule stands.

**At review time.** Flag violations of PolicyRules _except_ where an Exception covers them. A violation with no Exception is a problem; one covered by an Exception is recorded and moved on.

**At query time.** Retrieve by `applies_to` and `exception_to`. Present grouped by the rule being excepted.

**For expired Exceptions.** An Exception with `review_by` in the past has expired warranty. Agents should warn when relying on it and flag it as stale.

---

## Exception Health Signals

A high volume of exceptions against a single PolicyRule is a signal, not a problem to suppress. If the ratio of approved exceptions to actual violations approaches 1:1, the rule is probably too rigid—it should be revised or its enforcement tier downgraded, rather than maintained through a growing stack of exceptions.[^iso-ratio] Conversely, a PolicyRule with zero exceptions over a long period is either perfectly calibrated or not being enforced.

Review exceptions in aggregate, not just individually. Patterns worth watching:

- **Clustering by scope.** Many exceptions for the same service or domain suggests the rule doesn't fit that context. Consider a scoped revision.
- **Clustering by justification.** Identical justifications across exceptions means the rule has a structural gap.
- **Chronic renewals.** An exception renewed three or more times without change in conditions is a permanent deviation wearing a temporary label. Convert it to a PolicyRule amendment or supersession.

---

## Common Mistakes

**Filing an Exception when you wanted a Decision.** No PolicyRule to point at = not an Exception.

**Thin justification.** "Required by business" or "legacy system" is a label, not a justification.

**Missing compensating controls.** Removing protection without describing replacement is unsafe. Even if the answer is "risk accepted," say so explicitly.

**No review date.** A permanent Exception is a shadow rule. Force a review date.

**Scope creep.** Exception granted for one service silently applied to three. Make scope boundaries explicit with negative statements.

**Approver mismatch.** `approved_by` doesn't match the PolicyRule's `exception_authority`.

**Stacking Exceptions.** Excepting an Exception instead of superseding it.

**Exceptions for rules marked `exceptions_allowed: false`.** Either wrong rule or the rule needs to change, not be excepted.

---

## Notes

[^togaf-governance]: TOGAF's architecture governance framework requires that non-compliant changes receive formal **dispensation** from the Architecture Board. TOGAF's Compliance Assessment grades each review on a four-point scale — _Compliant_, _Compliant with Observations_, _Non-compliant_, and _Non-compliant with Dispensation_ — and an Exception record is the memory bank's durable representation of that last grade. Exceptions are a core governance artifact in the TOGAF model, not an edge case. See The Open Group. ["Architecture Governance"](https://pubs.opengroup.org/architecture/togaf9-doc/arch/chap44.html) (TOGAF Standard, Version 9.2, Chapter 44) and ["Architecture Compliance"](https://pubs.opengroup.org/architecture/togaf91-doc/arch/chap48.html) (Chapter 48, which defines the assessment grades). Both require a free Open Group account.

[^iso27001]: ISO/IEC 27001:2022 Annex A 5.1 requires that information security policies define exception handling procedures, including who can authorize deviations and how they are documented. See [ISMS.online: Annex A 5.1 — Information Security Policies](https://www.isms.online/iso-27001/annex-a-2022/5-1-information-security-policies-2022/).

[^iso-ratio]: The ISC2 recommends monitoring the ratio of approved exceptions to policy violations as a governance health metric. A 1:1 ratio suggests the policy is too rigid and encourages staff to bypass controls rather than comply. See ISC2. (2021). ["Exceptions to Security Policy — What Are They and How to Deal with Them?"](https://blog.isc2.org/isc2_blog/2021/02/exceptions-to-security-policy.html)

[^shadow-it]: "Shadow rules"—informal deviations that accumulate invisibly—are the governance analog of shadow IT, where unauthorized tools proliferate because formal channels are too rigid or slow. See [Wikipedia: Shadow IT](https://en.wikipedia.org/wiki/Shadow_IT); [Splunk: Shadow IT & How to Manage It](https://www.splunk.com/en_us/blog/learn/shadow-it.html).
