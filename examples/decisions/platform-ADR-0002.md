---
# --- Required -------------------------------------------------------
id: platform-ADR-0002
uuid: b2c3d4e5-f6a7-4b8c-9d0e-1f2a3b4c5d6e
memory_type: Decision
title: Adopt gRPC for internal service-to-service communication
status: accepted
owners:
  - role: platform-architect
    name: platform-architecture-guild

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2025-06-01
effective_to: null

applies_to:
  services: []
  domains: [platform, identity, builds, artifacts]
  systems: []

tags: [grpc, protobuf, api, service-communication, standards]

source_refs:
  - type: meeting
    ref: 'Platform ARB 2025-05-28'
  - type: ticket
    ref: 'PLAT-0892'

# --- Decision-specific fields ---------------------------------------
decision_question: 'What protocol should new services use for synchronous internal service-to-service communication?'
decision_outcome: 'gRPC with Protocol Buffers for all new internal synchronous communication. REST remains acceptable for public-facing APIs and third-party integrations.'

alternatives_considered:
  - option: 'REST/JSON over HTTP/1.1'
    reason_rejected: 'No schema enforcement between services; contract drift has caused three production incidents; no native streaming support.'
  - option: 'GraphQL federation'
    reason_rejected: 'Wrong level of abstraction for service mesh communication; adds n+1 query complexity without solving the schema drift problem.'
  - option: 'Apache Thrift'
    reason_rejected: 'Smaller ecosystem, weaker tooling compared to gRPC/protobuf; no meaningful advantage for our use case.'

decision_drivers:
  - 'Three production incidents in 2024 caused by implicit REST contract drift between services'
  - 'Schema-first development with protobuf catches breaking changes at compile time'
  - 'Bidirectional streaming required by the builds service for real-time log tailing'
  - 'gRPC natively integrates with the service mesh mTLS requirement (POL-0009)'
  - 'Code generation reduces boilerplate and enforces consistency across service clients'

approved_by:
  - 'Platform Architecture Review Board'

# --- Optional -------------------------------------------------------
implementation_status: in-progress
related:
  - uuid: b8c9d0e1-f2a3-4b4c-5d6e-7f8a9b0c1d2e
    relationship: constrained_by
---

## Context

Through 2024, services communicated over a mix of REST, hand-rolled HTTP clients, and a small number of internal GraphQL endpoints. There was no enforced contract mechanism. When a service changed a response shape, downstream consumers broke silently — the error appeared at runtime, not at build time. Three separate production incidents in 2024 were traced to this pattern, collectively requiring about 40 engineer-hours to diagnose and remediate.

The builds service also raised a requirements gap: it needed to stream build logs to the CLI in real time. The existing REST-polling approach introduced 2-3 second latency that degraded the developer experience on long builds.

## Decision

gRPC with Protocol Buffers is the standard for synchronous internal service-to-service communication. All new services exposing internal APIs must define them in `.proto` files, checked into a central `platform-protos` repository. Generated clients and server stubs are the only sanctioned way to call internal services.

REST remains the standard for public-facing APIs and third-party integrations, where gRPC's binary framing and protobuf encoding are inappropriate.

## Alternatives considered

**REST/JSON** was not ruled out on technical grounds alone — it has worked at larger scale than the platform currently operates. The decisive factor was the observed cost of contract drift. REST's flexibility is also its liability: nothing prevents a service from quietly changing a field name or dropping a nullable. gRPC's generated clients make breaking changes a compile error.

**GraphQL federation** was proposed to solve the schema problem while preserving JSON's readability. After evaluation, the ARB concluded that federation is a query pattern — designed for data graph aggregation — not a service communication protocol. It adds the N+1 resolver problem without solving anything gRPC doesn't solve more simply.

## Consequences

**Positive:** Schema drift incidents should approach zero for services that have migrated. The `platform-protos` repo becomes the machine-readable contract registry that was previously absent.

**Negative:** Four existing services communicate over REST internally. Each will need a migration plan. Services using the gRPC streaming API for log tailing must maintain backward compatibility with REST polling for a transition period.

**Neutral:** gRPC does not replace the message-passing patterns (Kafka topics) used for asynchronous workflows. This decision is scoped to synchronous RPC only.

## Related records

[POL-0009](../rules/platform-POL-0009.md) requires mutual TLS for all internal communication. gRPC's integration with the service mesh satisfies this requirement by default; services that implement gRPC correctly are in compliance with POL-0009 without additional configuration.
