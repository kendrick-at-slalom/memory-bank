---
id: platform-POL-0005
uuid: 8a2f1e4c-7b9d-4c3a-8e5f-1d2a3b4c5d6e
memory_type: PolicyRule
title: Postgres is the standard relational store across platform and commerce
status: proposed
owners:
  - role: platform-architect
    name: platform-architecture
applies_to:
  domains: [platform, commerce]
  systems: []
tags: [persistence, postgres, consolidation]
effective_from: 2026-03-29
source_refs:
  - type: adr
    ref: docs/adrs/0011-postgres-everywhere.md

rule_statement: "All new services in the platform and commerce domains use Postgres as their relational store unless they have a documented Exception."
enforcement: required
rationale: "Consolidation reduces tooling overhead, simplifies on-call context-switching, and shortens the learning curve for new engineers."
scope_of_application: "Applies to relational data stores in new services. Does not apply to non-relational workloads (key-value, search, time-series) or to existing services on other engines until they hit a major refactor."
exceptions_allowed: true
exception_authority:
  - "Platform Architecture"
review_cadence: annual
---

## Statement

All new services in the platform and commerce domains use Postgres as their relational store. Existing services on other engines stay where they are until they hit a major refactor, at which point they move to Postgres. Waivers require a documented Exception with a review date.

## Rationale

Service proliferation across MySQL, MariaDB, Postgres, and MS SQL has cost us in tooling overhead, on-call context-switching, and onboarding time. Consolidation on a single engine pays for itself within a few quarters.

## Scope

In: relational stores in new services within `platform` and `commerce` domains.

Out: non-relational workloads (covered by separate decisions per workload type), existing services on other engines (grandfathered until major refactor).

## Enforcement

Reviewed at architecture review for any new service. Holdouts flagged at quarterly platform health review.
