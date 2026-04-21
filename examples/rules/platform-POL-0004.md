---
# --- Required -------------------------------------------------------
id: platform-POL-0004
uuid: a7b8c9d0-e1f2-4a3b-4c5d-6e7f8a9b0c1d
memory_type: PolicyRule
title: All services must use PostgreSQL for durable persistence
status: accepted
owners:
  - role: platform-architect
    name: platform-architecture-guild

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2025-04-01
effective_to: null

applies_to:
  services: []
  domains: [platform, identity, builds, artifacts]
  systems: []

tags: [postgresql, persistence, database, standards, required]

source_refs:
  - type: document
    ref: 'Platform Engineering Standards v1.1'
  - type: meeting
    ref: 'Platform ARB 2025-03-25'

# --- PolicyRule-specific fields -------------------------------------
rule_statement: 'All services requiring durable persistence must use PostgreSQL 15 or later as their primary datastore.'
enforcement: required

rationale: >
  Standardizing on PostgreSQL reduces operational surface area (one backup strategy, one monitoring
  stack, one on-call knowledge base), enables shared platform tooling, and allows the platform team
  to provide a managed database offering. The cost of supporting multiple persistence technologies
  at our current team size is not justified by the flexibility gained. See ADR-0001 for the full
  history of this decision.

scope_of_application: >
  Applies to all new services in the platform, identity, builds, and artifacts domains requiring
  durable data storage. Existing services on other datastores are grandfathered at their current
  state and must adopt PostgreSQL at their next major version boundary. Does not apply to caching
  layers, time-series telemetry data, or transient data with TTL-based expiry (those may use
  purpose-appropriate stores with explicit ARB approval).

exceptions_allowed: true

exception_authority:
  - role: platform-architect
    name: platform-architecture-guild

review_cadence: annual

# --- Optional -------------------------------------------------------
policy_version: '1.1'
related:
  - uuid: a1b2c3d4-e5f6-4a7b-8c9d-0e1f2a3b4c5d
    relationship: derived_from
---

## Rule

All services requiring durable persistence must use PostgreSQL 15 or later.

"Durable persistence" means data that must survive a service restart. Caches, in-memory state, and data with explicit TTL-based expiry are not covered by this rule — see `scope_of_application` for precise boundaries.

## Why this rule exists

The platform's engineering standards audit in late 2024 found four different relational databases in production, each with its own operational requirements. The team supporting these systems couldn't maintain expertise across all four, and the resulting gaps caused at least one backup failure that went unnoticed for seven months.

PostgreSQL was selected as the standard in [ADR-0001](../decisions/platform-ADR-0001.md). This policy codifies that decision as a forward-looking constraint — not just a historical record of what was chosen, but a rule that shapes future choices.

## What compliance looks like

A new service is compliant if:

- It uses the platform-managed PostgreSQL offering (preferred), or
- It self-manages a PostgreSQL 15+ instance with backup, monitoring, and recovery documented in its runbook.

A service is non-compliant if it introduces a net-new non-Postgres database for durable storage without an approved Exception.

## Exception process

Exceptions are allowed but must be approved by the Platform Architecture Guild. To file an Exception:

1. Document why PostgreSQL cannot meet the requirement (load test results, latency benchmarks, or feature gaps are appropriate evidence).
2. Scope the Exception narrowly — a single service, a single data category.
3. Include compensating controls that address the operational risk of diverging from the standard.
4. Attach a review date (exceptions are not permanent).

See [EXC-0002](../exceptions/platform-EXC-0002.md) for an example of a well-structured Exception against this rule.

## Grandfathering

Services on MySQL, CockroachDB, or SQLite that pre-date this policy are grandfathered. They are not required to migrate immediately but must adopt PostgreSQL at the next major version boundary. The platform team maintains a registry of grandfathered services and their expected migration timelines.
