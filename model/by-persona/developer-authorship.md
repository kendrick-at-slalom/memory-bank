# Developer Authorship Guide

This guide walks through when you author memory bank records as a developer, and what each type looks like in your voice.

For the schema itself, see the parent files (`01`–`06`). This guide assumes you've skimmed the [model README](../README.md) at least once.

## When you author a record

Developer work produces records at these moments:

| Moment | Record type |
| --- | --- |
| You made a library, tool, or implementation-pattern choice that affects other developers or other services | **Decision** |
| You learned a gotcha, confirmed a performance characteristic, or discovered a vendor behavior that other developers need to know | **Context** |
| Your team codified a practice as the way we do things in this scope — e.g. "all outbound HTTP calls use the retry-backoff library" | **PolicyRule** |
| You accepted a workaround that deviates from a team practice or enterprise standard — e.g. this service logs in a legacy format because of framework constraints that are being retired | **Exception** |

The common thread: Decisions are implementation choices with follow-on reach. Contexts are the gotchas and characteristics that live in your head and need to live somewhere else. PolicyRules are team practices you'd want a new joiner (or an agent generating code) to follow without being told each time. Exceptions are the "we know, we have a reason" records that keep your codebase honest about its compromises.

The bar for writing a record: would a future teammate (or an agent working on your service) benefit from knowing this without reading the code? If yes, write it. Records that just restate what the code already says are noise.

## Field-fill cheat sheet

The schema defines each field; this table shows how you as a developer naturally phrase them.

| Field | What you put here |
| --- | --- |
| `decision_question` | The implementation choice you were making. "Which HTTP client library for outbound calls in the fulfillment-service?" |
| `decision_outcome` | The choice, one sentence. "Adopted `reqwest` with the retry-backoff middleware." |
| `alternatives_considered` | Libraries, patterns, or frameworks you looked at with one-sentence rejection reasons. |
| `decision_drivers` | Practical constraints that forced the choice. Existing infra, team skills, PolicyRules you inherit, performance needs, license compatibility. |
| `rule_statement` | The team practice stated imperatively. "All outbound HTTP calls use the retry-backoff client wrapper." |
| `fact_statement` | The operational fact in one paragraph. "The fulfillment-service has a known 200ms p99 latency spike during handoff windows between 02:00 and 02:15 UTC." |
| `exception_to` | UUID of the PolicyRule or team practice being deviated from. Required. |
| `justification` | Why the workaround exists. Framework limitation, legacy constraint, runtime environment specific, migration in progress. |
| `applies_to` | The scope of the record. Services, modules, environments. |
| `tags` | Topical labels. `http-client`, `retry`, `latency`, `legacy-framework`. |

## Worked examples

### Decision (an implementation pattern choice)

```yaml
---
id: fulfillment-DEV-0051
memory_type: Decision
title: Adopt reqwest + retry-backoff for outbound HTTP in fulfillment-service
status: accepted
owners:
  - role: tech-lead
    name: fulfillment-team
applies_to:
  services: [fulfillment-service]
  domains: [fulfillment]
tags: [http-client, retry, reqwest, outbound]
effective_from: 2026-04-05

decision_question: "Which HTTP client library should the fulfillment-service use for outbound calls?"
decision_outcome: "reqwest with the retry-backoff middleware as the default client wrapper."
alternatives_considered:
  - option: "hyper directly"
    reason_rejected: "Lower-level; team would re-implement retry/backoff patterns already handled by reqwest."
  - option: "ureq (sync client)"
    reason_rejected: "Sync-only; conflicts with the service's async runtime."
  - option: "isahc"
    reason_rejected: "Less community maintenance; slower CVE response in the past year."
decision_drivers:
  - "Team is already using reqwest elsewhere in the fulfillment stack; familiarity reduces onboarding cost"
  - "retry-backoff is a platform-approved library per POL-0031"
  - "Async-first, matches the service runtime"
approved_by:
  - role: tech-lead
    name: fulfillment-team
---
```

### Context (an operational gotcha)

```yaml
---
id: fulfillment-CTX-0044
memory_type: Context
title: Fulfillment-service p99 latency spikes during nightly handoff window
status: accepted
owners:
  - role: tech-lead
    name: fulfillment-team
applies_to:
  services: [fulfillment-service]
  environments: [production]
tags: [latency, performance, handoff-window, p99]
effective_from: 2026-02-20
last_verified: 2026-04-15

context_scope: service
fact_statement: >
  The fulfillment-service has a known 200ms p99 latency spike every night between 02:00 and 02:15 UTC
  during the order-reconciliation handoff window. The spike is caused by a synchronous reconciliation
  job that holds a connection-pool lock. The baseline p99 outside the window is ~45ms.
verifiability: >
  Visible on the fulfillment-service latency dashboard (Grafana: fulfillment/service-latency).
  Also reproducible by correlating with the reconciliation-job logs in Kibana.
assumptions:
  - "The reconciliation job continues to run at 02:00 UTC"
  - "The connection pool size is unchanged (currently 40)"
constraints:
  - "Services calling fulfillment during 02:00–02:15 should expect degraded latency and set timeouts accordingly"
  - "SLO calculations exclude the handoff window per the service's SLO definition"
  - "Proposals to move the reconciliation job should account for downstream dependencies surfaced in CTX-0045"
---
```

### PolicyRule (a team practice)

```yaml
---
id: fulfillment-POL-0012
memory_type: PolicyRule
title: All outbound HTTP calls in fulfillment use the platform retry-backoff wrapper
status: accepted
owners:
  - role: tech-lead
    name: fulfillment-team
applies_to:
  services: [fulfillment-service]
  domains: [fulfillment]
tags: [http-client, retry, resilience, team-practice]
effective_from: 2026-04-10

rule_statement: "All outbound HTTP calls from fulfillment-service code must go through the platform retry-backoff client wrapper. Direct use of raw HTTP clients is not permitted in production code paths."
enforcement: required
rationale: >
  Raw HTTP clients historically caused incident clusters around upstream flakiness. The retry-backoff
  wrapper encodes the team's agreed retry behavior (idempotent methods only, jittered exponential backoff,
  per-upstream caps). Centralizing the behavior lets us change retry policy in one place instead of auditing every call site.
scope_of_application: >
  Applies to all production code in the fulfillment-service repo. Does not apply to test utilities,
  local tooling, or one-off scripts in the `/scripts` directory. Applies to code generated by agents:
  if the agent proposes a raw HTTP call, reject the suggestion and route through the wrapper.
exceptions_allowed: true
exception_authority:
  - role: tech-lead
    name: fulfillment-team
review_cadence: annual
---
```

### Exception (a workaround)

```yaml
---
id: fulfillment-EXC-0008
memory_type: Exception
title: Legacy billing-adapter module exempted from retry-backoff wrapper requirement
status: accepted
owners:
  - role: tech-lead
    name: fulfillment-team
applies_to:
  services: [fulfillment-service]
  modules: [legacy-billing-adapter]
tags: [http-client, legacy, retirement-in-progress]
effective_from: 2026-04-10
effective_to: 2026-10-01

exception_to: 8f2e7b4a-1d9c-4e5f-3a2b-6c8d1e9f4a7b  # uuid of POL-0012
justification: >
  The legacy-billing-adapter module uses a vendor SDK that ships its own HTTP client. The client
  cannot be wrapped without forking the SDK. Retirement of this module is scheduled for 2026-10 when
  the replacement billing integration (DEV-0058) lands. Until then, the adapter retains its own retry
  behavior, which has been reviewed and found acceptable for its usage pattern (outbound calls to one
  well-known endpoint, low volume, per-request timeouts already in place).
approved_by:
  - role: tech-lead
    name: fulfillment-team
review_by: 2026-09-15
compensating_controls:
  - "Per-request timeout of 3s enforced in the adapter code"
  - "Alerting on adapter error rate via the fulfillment-alerts channel"
  - "Retirement tracking in the replacement billing integration Decision DEV-0058"
scope_boundary: >
  Applies only to the legacy-billing-adapter module inside fulfillment-service. Does not extend to
  any other legacy modules. Does not extend past the retirement date 2026-10-01.
---
```

## Common authorship mistakes, developer edition

**Writing Contexts that restate what the code already says.** "The service uses Postgres" is in the deployment manifest. "The service uses Postgres because we hit the MongoDB connection-pool ceiling in the 2025-11 incident" is a Context worth writing. Code says what; records say why.

**Writing a Decision for a trivial local choice.** A variable name, a small refactor, an internal function boundary: none of those need records. Decisions earn a record when the choice has reach beyond the immediate scope, meaning other teammates will encounter the pattern or an agent will need to follow it.

**PolicyRules that are really one-person preferences.** If the team hasn't agreed to the rule, it isn't one. A PolicyRule with `status: accepted` that surprised half the team when they read it is an indicator you wrote it too soon.

**Exceptions without retirement plans.** A workaround accepted "for now" that has no `review_by` and no linked retirement Decision will outlive the person who wrote it. Put the retirement plan in the compensating controls or link to the Decision that supersedes it.

**Stale Contexts.** The latency spike you captured in Context might go away when someone moves the reconciliation job. Set `last_verified` dates and update them when you confirm the fact is still true. A Context dated two years ago is suspect.
