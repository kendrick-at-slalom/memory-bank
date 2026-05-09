---
id: commerce-CTX-0019
uuid: 1c4d3e6f-9b2a-4e5c-8f7d-3a4b5c6d7e8f
memory_type: Context
title: Fulfillment cluster network is TCP-only with hard throughput cap
status: proposed
owners:
  - role: domain-architect
    name: fulfillment
applies_to:
  services: [fulfillment-service]
  domains: [fulfillment]
  systems: [plant-floor-network]
tags: [network, ot, throughput-constraint]
effective_from: 2026-04-15
source_refs:
  - type: meeting
    ref: "ARB 2026-04-15"

context_scope: service
fact_statement: "The fulfillment cluster sits behind a network appliance inherited from the OT side. The appliance permits TCP only (no UDP) and enforces a hard throughput cap that the engineering team cannot work around."
verifiability: "Confirmable via plant network ops. Last verified 2026-04-15 in ARB discussion (Devon, fulfillment lead)."
assumptions: []
constraints:
  - "No UDP traffic to or from fulfillment cluster"
  - "Throughput cap means high-frequency snapshot strategies are infeasible during peak"
---

## Fact

The fulfillment cluster sits behind a network appliance inherited from the OT side of the business. The appliance permits TCP only and enforces a hard throughput cap that engineering cannot relax.

## Background

Surfaced during the 2026-04-15 ARB discussion of order-replay snapshot strategy. The constraint was previously known by the fulfillment team but had not been written down. It affects at least three pending architecture decisions, including the order-replay snapshot strategy and any future fulfillment-side data export pattern.

## Verification

Plant network ops can confirm. Re-verify on any major plant infrastructure change.

## Implications

- Hourly snapshot strategies for the fulfillment cluster are infeasible during peak hours
- High-bandwidth integrations from fulfillment require either schedule-based windows or a different cluster
- Future architecture decisions touching fulfillment should reference this Context via `constrained_by`
