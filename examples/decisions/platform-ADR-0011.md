---
# --- Required -------------------------------------------------------
id: platform-ADR-0011
uuid: f6a7b8c9-d0e1-4f2a-3b4c-5d6e7f8a9b0c
memory_type: Decision
title: Upgrade at-rest encryption to AES-256-GCM
status: accepted
owners:
  - role: platform-security-lead
    name: platform-security-team

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2026-01-20
effective_to: null

applies_to:
  services: []
  domains: [platform, identity, builds, artifacts]
  systems: []

tags: [encryption, aes-256-gcm, security, data-at-rest, compliance]

source_refs:
  - type: meeting
    ref: 'Security Review 2026-01-15'
  - type: document
    ref: 'SOC 2 Type II Audit Finding SEC-2025-004'
  - type: document
    ref: 'Migration Runbook: AES-128-CBC to AES-256-GCM'
  - type: ticket
    ref: 'SEC-1102'

# --- Decision-specific fields ---------------------------------------
decision_question: 'Should the platform upgrade from AES-128-CBC (ADR-0003) to a stronger encryption standard to satisfy the 2025 SOC 2 Type II audit finding?'
decision_outcome: 'AES-256-GCM replaces AES-128-CBC as the platform encryption standard for all new data at rest. Existing data encrypted with AES-128-CBC must be migrated within 90 days of this decision per the migration runbook.'

alternatives_considered:
  - option: 'Retain AES-128-CBC with a documented risk acceptance'
    reason_rejected: 'SOC 2 audit finding requires remediation, not acceptance. Enterprise customer contracts include encryption minimums that AES-128-CBC no longer satisfies.'
  - option: 'ChaCha20-Poly1305'
    reason_rejected: 'Technically sound, but AES-256-GCM has broader hardware acceleration support in the cloud environment and better familiarity across the security team.'
  - option: 'AES-256-CBC'
    reason_rejected: 'CBC mode does not provide authenticated encryption. The audit finding specifically cited the absence of authentication as a gap; switching to AES-256-CBC would not address it.'

decision_drivers:
  - '2025 SOC 2 Type II audit flagged AES-128-CBC as inconsistent with NIST SP 800-57 Rev 5 (2020) for data with >10 year sensitivity window'
  - 'Three enterprise customer contracts include encryption minimums specifying AES-256 or equivalent'
  - 'AES-256-GCM provides authenticated encryption — it detects tampering in addition to providing confidentiality'
  - 'GCM mode has equivalent or better performance to CBC at AES-NI hardware acceleration levels present in target cloud infrastructure'

approved_by:
  - 'Platform Security Review Board'
  - 'Platform Architecture Review Board'

# --- Optional -------------------------------------------------------
implementation_status: in-progress
supersedes:
  - c3d4e5f6-a7b8-4c9d-0e1f-2a3b4c5d6e7f
---

## Context

In Q3 2025, the platform completed its first SOC 2 Type II audit. Finding SEC-2025-004 noted that the encryption standard established in [ADR-0003](./platform-ADR-0003.md) — AES-128-CBC — was inconsistent with NIST SP 800-57 Rev 5 guidance for data with retention periods exceeding ten years. Build artifacts and user account data both fall into this category. The finding also noted that CBC mode provides confidentiality but not authentication: an attacker with write access to the storage layer can modify ciphertext without detection.

Separately, three enterprise customer contracts were found to include encryption minimums requiring AES-256 or equivalent. The AES-128 standard was in violation of these terms.

## Decision

AES-256-GCM is the new encryption standard for all data at rest. The platform `platform-crypto` library has been updated to use AES-256-GCM by default. New services must use the updated library. Existing services must re-encrypt their data stores within 90 days using the migration runbook (referenced in source_refs).

GCM's authentication tag requirement means the encryption library must be used with an associated data (AD) parameter that binds each ciphertext to its storage location. This is a breaking API change for `platform-crypto` — see the migration guide for the updated interface.

This decision supersedes [ADR-0003](./platform-ADR-0003.md). ADR-0003 is preserved for audit trail.

## Alternatives considered

**Retaining AES-128-CBC with a risk acceptance** was briefly considered. A risk acceptance was not viable: the audit finding requires a remediation plan with a defined timeline, and three customer contracts are in literal non-compliance. A risk acceptance would not resolve either constraint.

**AES-256-CBC** would address the key length issue but not the authentication gap. The audit finding explicitly cited the absence of authenticated encryption. Switching to a longer key with CBC mode would require a second remediation cycle. GCM was the minimal change that addresses both findings at once.

**ChaCha20-Poly1305** is technically equivalent to AES-256-GCM for this use case and was considered seriously. The deciding factors were operational: AES-NI hardware acceleration is present on all target cloud instance types and reduces AES-256-GCM's computational overhead to near-zero. The security team's existing expertise is with AES. ChaCha20-Poly1305 has no meaningful advantage in this environment.

## Consequences

**Positive:** Platform is in compliance with the SOC 2 audit finding, NIST SP 800-57 Rev 5, and enterprise customer contract terms. Authenticated encryption detects ciphertext tampering — a meaningful additional security property.

**Negative:** All services with existing encrypted data stores must execute the 90-day migration. This is approximately 11 services. The `platform-crypto` API change is breaking; services must update their calls, not just their dependency version.

**Neutral:** The 90-day window was set to meet the audit remediation deadline. Services that cannot complete the migration in time must request a documented extension through the security team.

## Related records

This decision supersedes [platform-ADR-0003](./platform-ADR-0003.md). ADR-0003 has been updated with `status: superseded` and `superseded_by` pointing to this record.
