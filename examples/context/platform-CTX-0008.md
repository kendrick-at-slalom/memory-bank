---
# --- Required -------------------------------------------------------
id: platform-CTX-0008
uuid: f2a3b4c5-d6e7-4f8a-9b0c-1d2e3f4a5b6c
memory_type: Context
title: Builds service operating with degraded observability during metrics pipeline migration
status: accepted
owners:
  - role: builds-lead
    name: builds-team

# --- Recommended ----------------------------------------------------
confidence: reviewed

effective_from: 2026-04-01
effective_to: 2026-06-30

applies_to:
  services: [builds-service]
  domains: [builds]
  systems: []

tags: [observability, metrics, builds, migration, time-bounded]

source_refs:
  - type: ticket
    ref: 'BUILDS-0528'
  - type: document
    ref: 'Builds Service Observability Migration Plan v1.0'

# --- Context-specific fields ----------------------------------------
context_scope: service

fact_statement: >
  The builds service is actively migrating from a legacy StatsD/Graphite metrics pipeline to
  the platform observability stack (OpenTelemetry + Prometheus). During the migration window
  (2026-04-01 through 2026-06-30), build metrics may have up to a 5-minute lag in the platform
  Grafana dashboards. The legacy Graphite dashboards remain available as the source of truth
  for operational metrics during this period.

# --- Recommended ----------------------------------------------------
verifiability: >
  Migration status tracked in BUILDS-0528. Confirmed lag by comparing timestamps between
  legacy Graphite and platform Grafana on 2026-04-15 — observed 3–5 minute offset during
  high-volume build periods. Old dashboards: graphite.internal/builds. New dashboards:
  platform-grafana/builds (preview — not yet authoritative).

assumptions:
  - 'Migration will complete by 2026-06-30; if delayed, this Context record must be updated with a new effective_to date'
  - 'Legacy Graphite pipeline remains operational for the duration of the migration window'
  - 'The 5-minute lag is specific to derived metrics (queue depth, P99 build duration); raw build status events propagate in real time via the events topic'

constraints:
  - 'On-call runbooks for the builds service must reference Graphite dashboards, not platform Grafana, during the migration window'
  - 'Incident detection for builds service should not rely solely on platform Grafana alerts during this period'
  - 'SLA measurements for builds service are taken from Graphite exports, not platform Grafana, until migration is complete'
  - 'Any agent generating monitoring configuration for the builds service should note the dual-pipeline state'

# --- Optional -------------------------------------------------------
fact_type: operational
confidence_of_fact: high
last_verified: 2026-04-15
---

## The situation

The builds team has been running on a legacy StatsD/Graphite metrics pipeline that predates the platform observability stack. The platform team is migrating all services to OpenTelemetry + Prometheus, with Grafana as the unified dashboard layer.

The migration is in progress. As of April 2026, the builds service is emitting metrics to **both** pipelines simultaneously, but the OpenTelemetry pipeline has a known ingestion lag of 3–5 minutes for derived metrics (queue depth, percentile durations) under high build volume. Raw event data (build started, build completed) propagates in real time via the Kafka topic, but the aggregated metrics are delayed.

## Why this matters

Agents and engineers working on the builds service during this window need to know:

1. **Do not use platform Grafana as the authoritative monitoring source for the builds service until after 2026-06-30.** The dashboards are present but may show stale data during peak periods.

2. **Incident runbooks should reference Graphite directly.** The legacy dashboards at `graphite.internal/builds` are the source of truth for operational data during the migration window.

3. **SLA measurements must come from Graphite exports.** If a customer raises a builds SLA concern during this period, measurements must be taken from the Graphite historical data, not from platform Grafana.

## What this does not affect

Raw build events — build queued, build started, build completed, build failed — propagate through the Kafka topic in real time and are not affected by the metrics lag. Services consuming the build events topic (e.g., notification service, artifact service) are operating normally.

The builds-service API response times and error rates are also measured by the API gateway's own instrumentation, which is not affected by this migration.

## Record expiry

This Context record has `effective_to: 2026-06-30`. If the migration does not complete by that date, the builds team must either:

- Update this record with a new `effective_to` date, or
- Create a new Context record reflecting the updated migration state.

If the migration completes before 2026-06-30, this record should be updated to `status: deprecated` and `effective_to` set to the actual completion date.
