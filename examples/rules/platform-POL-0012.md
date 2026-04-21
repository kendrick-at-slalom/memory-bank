---
# --- Required -------------------------------------------------------
id: platform-POL-0012
uuid: c9d0e1f2-a3b4-4c5d-6e7f-8a9b0c1d2e3f
memory_type: PolicyRule
title: All feature experimentation must use the platform experimentation service
status: proposed
owners:
  - role: platform-product-lead
    name: platform-product-team

# --- Recommended ----------------------------------------------------
confidence: draft

effective_from: null
effective_to: null

applies_to:
  services: []
  domains: [platform, identity, builds, artifacts]
  systems: []

tags: [experimentation, feature-flags, a-b-testing, proposed]

source_refs:
  - type: ticket
    ref: 'PLAT-1147'
  - type: document
    ref: 'Experimentation Platform RFC v0.3 (draft)'

# --- PolicyRule-specific fields -------------------------------------
rule_statement: 'All feature experiments — A/B tests, multivariate tests, and canary rollouts — must be configured through the platform experimentation service. Experiments must not be implemented by hardcoding flag values, using environment variables, or maintaining per-service feature tables.'
enforcement: recommended

rationale: >
  Three teams are currently running separate feature flag implementations: two custom in-memory
  flag tables and one commercial feature flag SaaS subscription. None are connected to the platform
  analytics pipeline, so experiment results cannot be measured consistently. Two are updated by
  direct database writes in production (no review, no audit trail). Centralizing on the
  platform experimentation service provides consistent measurement, auditability, and rollback
  capabilities. The `recommended` enforcement is appropriate at this stage because the platform
  service is not yet feature-complete; teams will be required to migrate once the service reaches
  GA.

scope_of_application: >
  Applies to all product-facing feature experiments across all platform domains. Does not apply
  to operational flags (e.g., kill switches for known-broken code paths) or infrastructure
  rollout controls — those are handled by the deployment pipeline and are not subject to this rule.

exceptions_allowed: true

exception_authority:
  - role: platform-product-lead
    name: platform-product-team

review_cadence: quarterly

# --- Optional -------------------------------------------------------
policy_version: '0.3-draft'
---

> **Status: proposed.** This PolicyRule is under active discussion and has not been accepted by the ARB. It should not be treated as a constraint on current work. It is published here so teams can review and comment before the formal review on 2026-05-15.
>
> Open questions are listed at the bottom of this record.

## Background

The platform currently has no unified feature experimentation capability. As of April 2026:

- The identity team uses an in-memory flag table loaded from a YAML file at startup. Changes require a service restart.
- The builds team uses environment variables set during deployment for feature gating. There is no mechanism to target flags to specific users or cohorts.
- The artifacts team purchased a commercial feature flag SaaS. The subscription is personal (paid by one engineer) and is not integrated with the platform analytics pipeline.

None of these approaches produce measurements that can be compared or aggregated. The product team cannot answer "what was the experiment rollout rate last Tuesday" consistently across features.

The platform has been building an internal experimentation service (see RFC v0.3) that will provide: flag definition via config-as-code, percentage and cohort rollout targeting, consistent assignment (same user always gets the same variant), and automatic logging to the analytics pipeline.

## Proposed rule

Once the platform experimentation service reaches GA (projected Q3 2026), all new experiments must be created in it. Teams with existing flag implementations must migrate within 90 days of GA.

During the pre-GA period, the rule is `enforcement: recommended`. The intent is: don't build new custom flag implementations, but you are not required to migrate existing ones until GA.

## What this does not cover

**Operational kill switches** — flags used to disable broken code paths or rate-limit external API calls — are not covered by this rule. These are deployment-layer controls, not product experiments.

**Dark launches** — deploying code to production before it is user-visible — are allowed without going through the experimentation service, as long as no variant is being measured against a control.

## Open questions (to be resolved before ARB review)

1. Should `enforcement` be upgraded to `required` immediately (for new experiments only, not migrations), or stay `recommended` until GA? The product team proposes `recommended` now, `required` after GA.
2. What is the process for an Exception when the platform service does not yet support a required targeting capability (e.g., geographic rollout)?
3. Does this rule apply to the SDK developer documentation site (which has its own separate feature flag implementation)?

## Review date

ARB review is scheduled for **2026-05-15**. Teams should raise objections or provide feedback in PLAT-1147 before that date.
