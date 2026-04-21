---
# --- Required -------------------------------------------------------
id: platform-ADR-0003
uuid: c3d4e5f6-a7b8-4c9d-0e1f-2a3b4c5d6e7f
memory_type: Decision
title: Use AES-128-CBC for data-at-rest encryption
status: superseded
owners:
  - role: platform-security-lead
    name: platform-security-team

# --- Recommended ----------------------------------------------------
confidence: approved

effective_from: 2023-11-15
effective_to: 2026-01-20

applies_to:
  services: []
  domains: [platform, identity, builds, artifacts]
  systems: []

tags: [encryption, aes, security, data-at-rest]

source_refs:
  - type: meeting
    ref: 'Security Review 2023-11-10'
  - type: document
    ref: 'Security Standards v1.4'

# --- Decision-specific fields ---------------------------------------
decision_question: 'What encryption standard should be used for data at rest across platform services?'
decision_outcome: 'AES-128-CBC with PKCS#7 padding, using a key management service for key rotation. DEPRECATED — see platform-ADR-0011.'

alternatives_considered:
  - option: 'AES-256-GCM'
    reason_rejected: 'Evaluated but considered over-engineering at the time; AES-128 was considered sufficient for NIST guidelines in effect in 2023.'
  - option: 'ChaCha20-Poly1305'
    reason_rejected: 'Lower hardware acceleration support in the target cloud environment at time of decision.'

decision_drivers:
  - 'NIST SP 800-57 guidance current in 2023 accepted AES-128 for data with <10 year sensitivity window'
  - 'Existing KMS supports AES-128 key rotation without additional configuration'
  - 'Simpler CBC mode implementation reduced risk of implementation error compared to newer AEAD modes'

approved_by:
  - 'Platform Security Review Board'

# --- Optional -------------------------------------------------------
implementation_status: complete
superseded_by: f6a7b8c9-d0e1-4f2a-3b4c-5d6e7f8a9b0c
---

## Context

> **This record is superseded.** See [platform-ADR-0011](./platform-ADR-0011.md) for the current encryption standard. This record is preserved for institutional memory and audit trail only.

In 2023, the platform established its first formal encryption standard for data at rest. The scope covered all services storing personally identifiable information, financial data, or build artifacts. At the time, AES-128-CBC was widely accepted under NIST SP 800-57 Part 1 for data with a confidentiality window of less than ten years.

## Decision

AES-128-CBC with PKCS#7 padding was selected as the encryption standard. Key management was delegated to the platform KMS, with mandatory 90-day key rotation. Implementation was provided as a shared library (`platform-crypto`) to reduce the risk of teams implementing encryption incorrectly.

## Alternatives considered

AES-256-GCM was evaluated and noted in the original review as a more conservative choice. The ARB at the time determined that the additional computational overhead and implementation complexity of GCM's authentication tag were not justified by the threat model. This proved to be incorrect in hindsight — the 2025 compliance audit identified the absence of authenticated encryption (GCM provides both encryption and authentication; CBC does not) as a gap.

## Consequences

This standard served the platform for approximately 26 months before being superseded. The transition to ADR-0011 requires re-encrypting data in existing stores, which is addressed in the migration runbook referenced in ADR-0011's source references.

## Related records

[platform-ADR-0011](./platform-ADR-0011.md) supersedes this record. The supersession was triggered by a 2025 SOC 2 Type II audit finding that AES-128-CBC without authentication was inconsistent with current NIST guidance and contractual obligations to enterprise customers.
