# OSCAL Controls Export — Research

## OSCAL Model Overview

OSCAL (Open Security Controls Assessment Language) has five main document models defined
in the NIST SP 800-53 framework:

| Model | Acronym | Purpose |
|-------|---------|---------|
| System Security Plan | SSP | Describes a system's security controls implementation |
| Component Definition | CD | Defines reusable components with control implementations |
| Assessment Plan | AP | Defines how to assess controls |
| Assessment Results | AR | Records assessment outcomes |
| Plan of Action & Milestones | POA&M | Tracks remediation tasks |

**Best fit for VOIS:** Component Definition and SSP. Components map to VOIS actors/agents,
and control implementations map to VOIS controls.

## VOIS → OSCAL Mapping

### Component Definition

| VOIS Concept | OSCAL Element |
|-------------|---------------|
| Actor | `component` with type `software` or `service` |
| Agent | `component` with type `software` |
| Authority | `component` with type `service` + `party` in metadata |
| DigitalObject | `capability` or implemented under a component |
| VaultedObject | `component` with custom props or `implementation-status` |
| Control | `control-implementation` + `implemented-requirement` |
| Proof | `validation` or `evidence` within an assessment |
| Event | Action within `control-implementation` → `by-component` |
| Risk | `risk` in POA&M or `finding` in assessment |

### SSP Mapping

| VOIS Concept | SSP Section |
|-------------|-------------|
| System boundaries | `system-implementation` → `component` with boundary props |
| Actors/users | `system-implementation` → `user` |
| Authorities | `system-implementation` → `component` with authorisation props |
| Controls | `control-implementation` → implemented-requirements |
| Evidence | `control-implementation` → `implementation-status` + remarks |
| Data objects | `system-information` → `information-type` |

## OSCAL JSON Structure Basics

```json
{
  "component-definition": {
    "uuid": "urn:uuid:...",
    "metadata": {
      "title": "VOIS Insurance Claim Components",
      "last-modified": "2026-07-30T00:00:00Z",
      "version": "0.1.0",
      "parties": [
        {
          "uuid": "urn:uuid:auth-001",
          "type": "organization",
          "name": "Insurer Authority",
          "props": [
            {"name": "vois-role", "value": "certifying_authority"},
            {"name": "seal-types", "value": "origin_seal,authority_seal,attestation"}
          ]
        }
      ]
    },
    "components": [
      {
        "uuid": "urn:uuid:agt-001",
        "type": "software",
        "title": "Claims Intake Agent",
        "description": "AI agent that receives claims and verifies policies",
        "props": [
          {"name": "vois-entity-type", "value": "agent"},
          {"name": "vois-delegated-by", "value": "urn:uuid:act-002"}
        ],
        "control-implementations": [
          {
            "uuid": "urn:uuid:ci-001",
            "source": "https://www.nist.gov/800-53/5.1",
            "description": "Control implementations for claims processing",
            "implemented-requirements": [
              {
                "uuid": "urn:uuid:ctl-001",
                "control-id": "ac-3",
                "description": "Policy Valid Before Processing",
                "props": [
                  {"name": "vois-control-type", "value": "verification_check"},
                  {"name": "vois-governance-rule", "value": "Claim must match active policy"}
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

## Example: Insurance Claim Controls in OSCAL

```json
{
  "component-definition": {
    "uuid": "urn:uuid:ins-claim-001",
    "metadata": {
      "title": "VOIS Insurance Claim Evidence Workflow",
      "last-modified": "2026-07-30T00:00:00Z",
      "version": "0.1.0",
      "roles": [
        {"id": "vois-actor", "title": "VOIS Actor"},
        {"id": "vois-authority", "title": "VOIS Authority"}
      ],
      "parties": [
        {"uuid": "urn:uuid:auth-001", "type": "organization", "name": "Insurer Authority"},
        {"uuid": "urn:uuid:auth-002", "type": "organization", "name": "Accredited Assessor Authority"},
        {"uuid": "urn:uuid:auth-003", "type": "organization", "name": "Industry Regulator"}
      ]
    },
    "components": [
      {
        "uuid": "urn:uuid:ctl-001",
        "type": "software",
        "title": "Policy Valid Before Processing",
        "description": "Verification check: claim must match active policy",
        "props": [
          {"name": "vois-control-type", "value": "verification_check"},
          {"name": "vois-governance-rule", "value": "Claim must match active policy"}
        ]
      },
      {
        "uuid": "urn:uuid:ctl-002",
        "type": "software",
        "title": "Evidence Must Be Sealed",
        "description": "Seal requirement: all evidence must carry origin seal",
        "props": [
          {"name": "vois-control-type", "value": "seal_requirement"},
          {"name": "vois-governance-rule", "value": "All evidence must carry origin seal"}
        ]
      }
    ]
  }
}
```

## Tools & Libraries

| Library | Language | Purpose |
|---------|----------|---------|
| oscal-js (nist-open-source) | TypeScript | OSCAL JSON processing utilities |
| @easygen/oscasl | TypeScript | OSCAL to HTML/PDF converters |
| trestle (via CLI) | Python | NIST Trestle — OSCAL management toolkit |
| OSCAL JSON Schema | — | Available from NIST at https://pages.nist.gov/OSCAL/ |

## Key NIST 800-53 Control IDs Relevant to VOIS

| Control ID | Name | VOIS Relevance |
|------------|------|----------------|
| AC-3 | Access Enforcement | Authority-based access controls |
| AU-2 | Audit Events | Event provenance logging |
| AU-3 | Content of Audit Records | Proof/evidence generation |
| CM-2 | Baseline Configuration | Blueprint as configuration baseline |
| IA-2 | Identification and Authentication | Actor/authority identity |
| SC-8 | Transmission Confidentiality | Zero-Exposure communication |
| SC-12 | Cryptographic Key Establishment | ITS key management |
| SI-7 | Software Integrity Verification | Digital seal verification |

## Limitations

1. **No VOIS-native concepts** — OSCAL has no native representation for possession, ITS
   proofs, digital seals, or Zero-Exposure infrastructure. All VOIS semantics must go
   in custom `props`.
2. **Heavyweight** — OSCAL JSON is deeply nested with UUIDs and references throughout.
   Generating valid OSCAL requires strict UUID bookkeeping.
3. **US-centric** — OSCAL is built around NIST 800-53. Non-US regulatory frameworks need
   custom control mappings.
4. **Assessment-first, not design-first** — OSCAL is designed for post-implementation
   assessment, not design-time blueprinting. Blueprints map better to SSP for system
   description and Component Definition for the authority/control model.
