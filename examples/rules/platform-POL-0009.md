---
# --- Required -------------------------------------------------------
id: platform-POL-0009
uuid: b8c9d0e1-f2a3-4b4c-5d6e-7f8a9b0c1d2e
memory_type: PolicyRule
title: All internal service communication must use mutual TLS
status: accepted
owners:
  - role: platform-security-lead
    name: platform-security-team

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2025-01-15
effective_to: null

applies_to:
  services: []
  domains: [platform, identity, builds, artifacts]
  systems: []

tags: [mtls, tls, security, service-mesh, required]

source_refs:
  - type: document
    ref: 'Platform Security Standards v2.0'
  - type: meeting
    ref: 'Security Review Board 2025-01-10'
  - type: document
    ref: 'SOC 2 CC6.6 Control Mapping'

review_by: 2027-01-15

# --- PolicyRule-specific fields -------------------------------------
rule_statement: 'All synchronous communication between services within the platform network must use mutual TLS (mTLS). Both client and server must present valid certificates issued by the platform certificate authority.'
enforcement: required

rationale: >
  Service-to-service communication without mutual authentication creates an implicit trust boundary
  at the network perimeter. Any service, container, or process that gains access to the internal
  network can call any other service without authentication. mTLS moves the trust boundary to
  the service identity level: a service must prove it holds a valid platform-issued certificate
  to communicate, regardless of network position. This is a control mapped to SOC 2 CC6.6
  (Logical and Physical Access Controls).

scope_of_application: >
  Applies to all synchronous RPCs between services within the platform Kubernetes clusters.
  Does not apply to: public-facing API endpoints (covered by TLS, not mTLS), communication
  with third-party services outside the platform network, or communication between components
  within a single service binary (in-process calls).

exceptions_allowed: true

exception_authority:
  - role: platform-security-lead
    name: platform-security-team

review_cadence: biennial

# --- Optional -------------------------------------------------------
policy_version: '2.0'
---

## Rule

All inter-service communication within the platform network must use mutual TLS. Both the calling service and the receiving service must present certificates issued by the platform certificate authority for each connection.

The platform service mesh provides mTLS automatically for services using the standard service mesh configuration. Services that communicate outside the mesh (e.g., direct TCP connections to databases or third-party APIs) must implement mTLS independently or document why it does not apply.

## Why this rule exists

A compromised container or misconfigured service on the internal network can reach any other service if network-level controls are the only trust boundary. In a shared Kubernetes cluster, lateral movement following a compromise is straightforward without service-level authentication.

mTLS ensures that both the caller and the callee are who they claim to be. A certificate issued by the platform CA is required to open a connection — network position alone is not sufficient. Combined with workload identity (certificate subjects are scoped to specific services and namespaces), this also creates an auditable record of which services are communicating, which supports the SOC 2 CC6.6 control requirement.

## What compliance looks like

- **Service mesh deployments:** Services deployed with the standard mesh sidecar configuration are compliant by default. No additional configuration is required.
- **Non-mesh deployments:** Services that for architectural reasons cannot use the mesh sidecar must configure mTLS directly using certificates provisioned through the platform certificate manager. These configurations must be reviewed by the security team at deployment.

## gRPC services

Services using gRPC (per [ADR-0002](../decisions/platform-ADR-0002.md)) satisfy this policy by default through service mesh integration. The gRPC client library is pre-configured to use the mesh certificate. No additional TLS configuration is needed at the application layer.

## Exceptions

Exceptions to this policy require approval from the Platform Security Lead. The bar for exceptions is higher than for other platform policies given the SOC 2 control mapping. Evidence must demonstrate either: (a) technical infeasibility, or (b) equivalent or better authentication control achieved through another mechanism. See the Exception type documentation for the expected fields.
