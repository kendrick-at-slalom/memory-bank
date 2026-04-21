---
# --- Required -------------------------------------------------------
id: platform-CTX-0005
uuid: e1f2a3b4-c5d6-4e7f-8a9b-0c1d2e3f4a5b
memory_type: Context
title: Identity service session validation has a 10ms p99 SLA
status: accepted
owners:
  - role: identity-lead
    name: identity-team

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2025-05-01
effective_to: null

applies_to:
  services: [identity-service]
  domains: [identity]
  systems: []

tags: [sla, latency, session, identity, performance]

source_refs:
  - type: document
    ref: 'Identity Service SLA Agreement v1.2'
  - type: document
    ref: 'Platform API Contract 2025'
  - type: ticket
    ref: 'IDENT-0190'

# --- Context-specific fields ----------------------------------------
context_scope: service

fact_statement: >
  The identity service must complete session validation — the full operation of verifying a
  session token and returning the associated user claims — within 10 milliseconds at the 99th
  percentile. This SLA is contractual: it is documented in the Platform API Contract and
  referenced in enterprise customer SLAs. Exceeding it degrades authentication latency for
  every platform service that calls the identity service on the request path.

# --- Recommended ----------------------------------------------------
verifiability: >
  Documented in Identity Service SLA Agreement v1.2, section 3.2. Measured via the
  `identity.session.validate.duration_ms` histogram in the platform observability stack.
  Dashboard: platform-grafana/identity-sla. Any P99 alert on this metric triggers an
  immediate on-call page.

assumptions:
  - 'The SLA applies under normal operating load; burst scenarios above 2x peak are governed by separate degradation policies'
  - "The identity service is the only service with a contractual session validation SLA; other services' use of session validation is subject to this SLA transitively"

constraints:
  - 'Datastores used for session lookup must be capable of meeting 10ms p99 under peak load before they can be adopted'
  - 'Any change to the session validation code path must include a load test demonstrating continued SLA compliance'
  - 'Architectural changes that add network hops to the session validation path require explicit SLA impact assessment'

# --- Optional -------------------------------------------------------
fact_type: operational
confidence_of_fact: high
last_verified: 2026-04-01
---

## The fact

Session validation must complete in 10ms p99. This includes the full roundtrip: receiving the token, looking up the session, verifying expiry and revocation status, and returning user claims to the caller.

This is not a target or a best-effort goal. It is a contractual commitment reflected in enterprise customer SLAs. If the identity service consistently exceeds 10ms p99, the platform is in breach of those contracts.

## Why this matters to other records

This Context is the environmental fact that motivates [EXC-0002](../exceptions/platform-EXC-0002.md) and [ADR-0007](../decisions/platform-ADR-0007.md). Without it, the decision to use Redis instead of PostgreSQL for session storage looks like a preference. With it, the decision makes sense: PostgreSQL cannot meet the SLA under peak load, and no amount of optimization closes the gap.

Agents should surface this fact whenever a change is proposed to the identity service's session validation path: adding middleware, changing the session store, adding a new claim source, or calling an external service during validation. Each of these adds latency.

## How this SLA was established

The 10ms figure was negotiated as part of the Platform API Contract during a 2025 enterprise customer onboarding cycle. The contract was informed by the identity team's benchmarks showing that <10ms session validation was achievable and sustainable.

Before the formal SLA, there was no documented performance requirement. The absence of a target contributed to gradual performance degradation over 2023–2024 as the validation code path accumulated small additions (each individually harmless) that collectively pushed P99 to 28ms by Q1 2025.

The SLA is now enforced via the `identity.session.validate.duration_ms` P99 alert. The dashboard is the authoritative measurement source.

## Monitoring and verification

The SLA is measured continuously. The P99 alert threshold is 9ms — below the contractual 10ms — to provide a warning window before breach. Any condition that causes the P99 alert to fire for more than five consecutive minutes is treated as an SLA incident.
