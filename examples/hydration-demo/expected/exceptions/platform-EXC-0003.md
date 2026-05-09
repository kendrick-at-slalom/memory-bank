---
id: platform-EXC-0003
uuid: 9b3c2f5d-8a0e-4d4b-9f6a-2e3b4c5d6e7f
memory_type: Exception
title: auth-service uses Redis for session cache, exempt from POL-0005
status: proposed
owners:
  - role: domain-architect
    name: platform-architecture
applies_to:
  services: [auth-service]
  domains: [platform]
tags: [redis, session-cache, latency]
effective_from: 2026-04-30
effective_to: 2026-10-15
source_refs:
  - type: pr
    ref: PR-1308
  - type: ticket
    ref: AUTH-204

exception_to: 8a2f1e4c-7b9d-4c3a-8e5f-1d2a3b4c5d6e
justification: "Session lookups are pure key-value reads at ~12k/sec with an 8ms p99 budget. Postgres-backed implementation cannot meet the budget under load after two sprints of tuning."
approved_by:
  - "@priya (Platform Architecture)"
  - "@platform-arch"
review_by: 2026-10-15
compensating_controls:
  - "Redis cluster has equivalent backup, monitoring, and secret-rotation tooling as Postgres clusters"
  - "Failover to Postgres on Redis unavailability (degraded latency, acceptable)"
  - "Session loss on Redis failure is acceptable; sessions re-issue on next auth"
scope_boundary: "Covers auth-service only. Does not authorize Redis for any other service; future Redis use cases require separate Exceptions or an amendment to POL-0005."
---

## Rule being excepted

POL-0005 (Postgres as the standard relational store across platform and commerce).

## Justification

Session lookups for `auth-service` run ~12k/sec at peak with an 8ms p99 budget set by the API gateway SLO. The Postgres-backed implementation hit the budget about 60% of the time after two sprints of tuning (connection pooling, prepared statements, table partitioning). Architecturally, session lookups have no relational shape; pushing them through a relational engine doesn't earn the cost.

## Compensating controls

See `compensating_controls` field.

## Scope

In: `auth-service` session cache.

Out: any other use of Redis in any other service. Future Redis cases require separate Exceptions or an amendment to POL-0005.

## Review and expiration

Review date 2026-10-15. At review, the team confirms either: (a) the exception remains valid and is renewed, (b) the underlying Postgres bottleneck is resolved and the exception lapses, or (c) Redis adoption has expanded enough that POL-0005 needs amending.
