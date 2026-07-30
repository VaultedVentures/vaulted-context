# W3C PROV Provenance Mapping for VOIS Blueprints

> Research document for the Vaulted Objects Infrastructure (VOI) Simulator
> Date: 2026-07-31
> Purpose: Map VOIS blueprint concepts to W3C PROV provenance model for interoperable evidence tracking

---

## 1. W3C PROV Overview

**W3C PROV** is a family of specifications for interoperable provenance interchange on the Web. It defines a conceptual data model (PROV-DM), an OWL2 ontology (PROV-O), a human-readable notation (PROV-N), XML schema (PROV-XML), and a JSON-LD serialization (PROV-JSONLD).

**Core definition**: Provenance records contain descriptions of the entities and activities involved in producing and delivering or otherwise influencing a given object. Provenance can be used for:
- Understanding how data was collected
- Determining ownership and rights over an object
- Making trust judgments about information
- Verifying that processes comply with requirements
- Reproducing how something was generated

**Namespace**: `http://www.w3.org/ns/prov#` (prefix `prov:`)

---

## 2. PROV Core Concepts

### 2.1 Entity
- **Definition**: A physical, digital, conceptual, or other kind of thing.
- **PROV-O class**: `prov:Entity`
- **Examples**: document, web page, chart, dataset, image, vaulted object, blueprint
- **Attributes**: identifier (URI), type, location, label, value, custom typed attributes
- **Visual**: oval in diagrams

### 2.2 Activity
- **Definition**: How entities come into existence and how their attributes change, often making use of previously existing entities.
- **PROV-O class**: `prov:Activity`
- **Examples**: writing, editing, compiling, signing, verifying, sealing, composing
- **Attributes**: identifier (URI), startTime, endTime, type, location, label
- **Visual**: rectangle in diagrams

### 2.3 Agent
- **Definition**: Someone or something that takes a role in an activity and can be assigned some degree of responsibility.
- **PROV-O class**: `prov:Agent`
- **Subtypes**: `prov:Person`, `prov:Organization`, `prov:SoftwareAgent`
- **Examples**: human operator, automated signing service, vault service, auditor
- **Visual**: pentagon in diagrams

### 2.4 Core Relations (Starting Point Terms)

| PROV Relation | Direction | Meaning | PROV-O Property |
|---|---|---|---|
| `wasGeneratedBy` | Entity → Activity | Entity was produced by the activity | `prov:wasGeneratedBy` |
| `used` | Activity → Entity | Activity made use of the entity | `prov:used` |
| `wasAttributedTo` | Entity → Agent | Entity is ascribed to the agent | `prov:wasAttributedTo` |
| `wasAssociatedWith` | Activity → Agent | Agent was associated with the activity | `prov:wasAssociatedWith` |
| `actedOnBehalfOf` | Agent → Agent | Delegation chain | `prov:actedOnBehalfOf` |
| `wasDerivedFrom` | Entity → Entity | One entity derived from another | `prov:wasDerivedFrom` |
| `wasInformedBy` | Activity → Activity | Communication between activities | `prov:wasInformedBy` |
| `wasRevisionOf` | Entity → Entity | Specialized derivation (revision) | `prov:wasRevisionOf` |
| `startedAtTime` | Activity → xsd:dateTime | Activity start time | `prov:startedAtTime` |
| `endedAtTime` | Activity → xsd:dateTime | Activity end time | `prov:endedAtTime` |
| `generatedAtTime` | Entity → xsd:dateTime | Entity generation time | `prov:generatedAtTime` |
| `invalidatedAtTime` | Entity → xsd:dateTime | Entity invalidation time | `prov:invalidatedAtTime` |
| `alternateOf` | Entity → Entity | Two descriptions of the same thing | `prov:alternateOf` |
| `specializationOf` | Entity → Entity | One entity is a specialization of another | `prov:specializationOf` |

### 2.5 Expanded Terms

| Term | Description |
|---|---|
| `prov:Bundle` | A named set of provenance descriptions (self-contained document) |
| `prov:Collection` | An entity that contains other entities |
| `prov:Plan` | A plan or recipe followed by an activity |
| `prov:Location` | A location (can be named, geospatial, etc.) |
| `prov:wasQuotedFrom` | Derivation via quotation |
| `prov:hadPrimarySource` | Derivation from a primary source |
| `prov:hadMember` | Collection membership |

### 2.6 Qualified Terms (for detailed annotations)

When binary relations need richer metadata (roles, times, plans, etc.), PROV-O provides qualification patterns using reified relationship classes:

| Qualified Class | Qualifies | Properties |
|---|---|---|
| `prov:Generation` | `wasGeneratedBy` | entity, activity, time, role |
| `prov:Usage` | `used` | activity, entity, time, role |
| `prov:Association` | `wasAssociatedWith` | activity, agent, plan, role |
| `prov:Attribution` | `wasAttributedTo` | entity, agent |
| `prov:Derivation` | `wasDerivedFrom` | generatedEntity, usedEntity, activity, usage, generation |
| `prov:Revision` | (subclass of Derivation) | — |
| `prov:Quotation` | (subclass of Derivation) | — |
| `prov:PrimarySource` | (subclass of Derivation) | — |
| `prov:Communication` | `wasInformedBy` | informed, informant |
| `prov:Delegation` | `actedOnBehalfOf` | delegate, responsible, activity |
| `prov:Start` | `wasStartedBy` | activity, trigger, starter |
| `prov:End` | `wasEndedBy` | activity, trigger, ender |
| `prov:Invalidation` | `wasInvalidatedBy` | entity, activity, time |

---

## 3. VOIS → PROV Mapping Table

### Core VOIS Concepts

| VOIS Concept | PROV Concept | Rationale |
|---|---|---|
| **Vaulted Object** | `prov:Entity` | A digital object whose provenance is tracked. Can be typed with a subtype (`vault:VaultedObject`) |
| **Blueprint** | `prov:Entity` + `prov:Plan` | Blueprints are both entities (documents) and plans (they define procedures). Dual classification. |
| **Blueprint Schema** | `prov:Entity` | The schema definition for a blueprint structure |
| **Blueprint Instance** | `prov:Entity` + `specializationOf` | An instance is a specialization of the blueprint |
| **Claim** | `prov:Entity` | A claim about object state, possession, or attribute |
| **Evidence Package** | `prov:Bundle` | A named set of provenance assertions packaging related claims and proofs |
| **Attestation Activity** | `prov:Activity` | The act of generating/sealing evidence |
| **Verification Activity** | `prov:Activity` | The act of verifying a claim or proof |
| **Sealing Activity** | `prov:Activity` | The act of applying an ITS seal |
| **Signing Service** | `prov:Agent` (`prov:SoftwareAgent`) | Automated agent performing sealing/signing |
| **Human Operator** | `prov:Agent` (`prov:Person`) | Human responsible for an action |
| **Auditor** | `prov:Agent` (`prov:Person`) | Verifier of provenance |
| **Vault Service** | `prov:Agent` (`prov:Organization`) | The service providing vault functionality |

### Relations

| VOIS Relation | PROV Relation | Description |
|---|---|---|
| Object is sealed | `wasGeneratedBy` (Sealing Activity) + `wasAttributedTo` (Signing Service) | The sealed object is generated by the sealing activity |
| Proof is generated | `wasGeneratedBy` (Attestation Activity) | The proof entity is generated by the attestation |
| Evidence uses claim | `used` | The attestation activity used the claim entity |
| Claim is attributed to actor | `wasAttributedTo` | The claim entity is attributed to the agent who made it |
| Blueprint is derived from schema | `wasDerivedFrom` | Blueprints are derived from their schema |
| Blueprint instance derives from blueprint | `wasDerivedFrom` | Instance inherits blueprint structure |
| Evidence package contains claim | `hadMember` | Bundle/collection membership |
| Actor delegates to service | `actedOnBehalfOf` | Human operator delegates to automated service |
| Claim is verified | `wasGeneratedBy` (Verification Activity) | The verification result is generated by a verify activity |
| Seal is attached | `wasGeneratedBy` (Sealing Activity) | Seal application as generation |
| Chronological ordering | `startedAtTime`, `endedAtTime`, `generatedAtTime` | Temporal ordering of activities and entities |

---

## 4. ITS Seals, Possession Proofs, and Zero-Exposure Verification in PROV

### 4.1 ITS Seals

An **ITS (Identity Trust System) Seal** is a cryptographic attestation binding an entity's state at a point in time to a trusted identity.

**PROV mapping**:

```
┌─────────────────────────────────────────────────────────┐
│  prov:Entity (VaultedObject)                            │
│    wasGeneratedBy → prov:Activity (SealingActivity)     │
│    wasAttributedTo → prov:Agent (SigningService)        │
│                                                         │
│  prov:Activity (SealingActivity)                        │
│    used → VaultedObject (pre-seal version)              │
│    used → PrivateSigningKey (as entity)                 │
│    wasAssociatedWith → SigningService                   │
│    startedAtTime → t1                                   │
│    endedAtTime → t2                                     │
│                                                         │
│  prov:Entity (SealCertificate)                          │
│    wasGeneratedBy → SealingActivity                     │
│    wasAttributedTo → SigningService                     │
│    wasDerivedFrom → VaultedObject                       │
│    generatedAtTime → t2                                 │
│    → cryptographic hash as prov:value                   │
└─────────────────────────────────────────────────────────┘
```

The seal itself becomes a `prov:Entity` with:
- `prov:type`: `vois:Seal`
- `vois:signatureAlgorithm`: ECDSA/Ed25519/etc.
- `vois:sealDigest`: hex-encoded digest
- `vois:sealTime`: ISO 8601 timestamp

### 4.2 Possession Proofs

A **Possession Proof** demonstrates that an agent held or controlled a vaulted object at a particular time.

**PROV mapping**:

```
entity(vois:possessionProof_001, [
  prov:type='vois:PossessionProof',
  vois:objectRef=vois:vaultedObject_42,
  vois:possessor=vois:agent_alice,
  vois:provenanceHash="abc123..."
])

wasGeneratedBy(vois:possessionProof_001, vois:attestation_001, 2026-07-31T10:00:00Z)

used(vois:attestation_001, vois:vaultedObject_42, -)
wasAssociatedWith(vois:attestation_001, vois:agent_alice, -)
wasAttributedTo(vois:possessionProof_001, vois:agent_alice)
```

- The PossessionProof entity is generated by an Attestation activity
- The Attestation activity uses the vaulted object and is associated with the possessor
- The proof is attributed to the agent who demonstrates possession

### 4.3 Zero-Exposure Verification

**Zero-Exposure Verification** means proving a property about a claim without revealing the underlying data. In PROV terms:

- The **Verification Activity** uses a **Commitment** (e.g., hash, Pedersen commitment, zk-SNARK proof) rather than the raw data
- The **Verification Result** is generated from the verification activity
- The verification attests that a property holds **without** the verifier ever accessing the raw claim

```
entity(vois:propertyCommitment_001, [
  prov:type='vois:CryptographicCommitment',
  vois:commitmentType='zkSnark',
  vois:circuitId='possession_verification_v1',
  vois:commitmentDigest="0x..."
])

entity(vois:verificationResult_001, [
  prov:type='vois:VerificationResult',
  vois:result='valid'
])

activity(vois:verify_001, 2026-07-31T11:00:00Z, 2026-07-31T11:00:05Z)

used(vois:verify_001, vois:propertyCommitment_001, -)
used(vois:verify_001, vois:verificationKey_001, -)
wasGeneratedBy(vois:verificationResult_001, vois:verify_001, -)
wasAssociatedWith(vois:verify_001, vois:auditor_bob, -)
```

**Key insight**: The raw `claim` entity is never referenced in the verification PROV trail — only the commitment is `used`. This achieves zero-exposure semantics while still building a complete provenance graph.

---

## 5. PROV-N Notation for the Claim Evidence Workflow

### 5.1 Basic PROV-N Syntax Reference

```
document
  prefix prov <http://www.w3.org/ns/prov#>
  prefix vois <http://vaulted.org/ns/vois#>
  prefix xsd <http://www.w3.org/2001/XMLSchema#>

  entity(vois:object_42, [prov:type='vois:VaultedObject'])
  entity(vois:claim_001, [prov:type='vois:Claim', vois:property='possession', prov:value="alice owns object_42"])
  activity(vois:attest_001, 2026-07-31T09:00:00Z, 2026-07-31T09:00:10Z)
  agent(vois:alice, [prov:type='prov:Person'])
  agent(vois:signingService, [prov:type='prov:SoftwareAgent'])

  used(vois:attest_001, vois:claim_001, -)
  wasGeneratedBy(vois:proof_001, vois:attest_001, -)
  wasAssociatedWith(vois:attest_001, vois:alice, -)
  wasAttributedTo(vois:claim_001, vois:alice)
endDocument
```

### 5.2 Full Claim Evidence Workflow (PROV-N)

```
document
  prefix prov <http://www.w3.org/ns/prov#>
  prefix vois <http://vaulted.org/ns/vois#>
  prefix xsd <http://www.w3.org/2001/XMLSchema#>

  /* === Entities === */
  entity(vois:blueprint_01, [prov:type='vois:Blueprint', prov:label="Asset Transfer Blueprint"])
  entity(vois:blueprintSchema_01, [prov:type='vois:BlueprintSchema'])
  entity(vois:claim_001, [
    prov:type='vois:Claim',
    vois:blueprintRef=vois:blueprint_01,
    vois:predicate='possesses',
    vois:subject='alice',
    vois:object='vois:object_42'
  ])
  entity(vois:possessionProof_001, [
    prov:type='vois:PossessionProof',
    vois:provenanceHash="e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
  ])
  entity(vois:seal_001, [
    prov:type='vois:Seal',
    prov:value="MEUCIQ...signature..."
  ])
  entity(vois:evidencePackage_001, [
    prov:type='vois:EvidencePackage'
  ])
  entity(vois:verificationResult_001, [
    prov:type='vois:VerificationResult',
    vois:result='valid',
    vois:verifiedAt="2026-07-31T10:00:00Z"^^xsd:dateTime
  ])

  /* === Activities === */
  activity(vois:createBlueprint_01, 2026-07-30T08:00:00Z, 2026-07-30T09:00:00Z)
  activity(vois:assertClaim_001, 2026-07-31T08:30:00Z, 2026-07-31T08:35:00Z)
  activity(vois:attestProof_001, 2026-07-31T09:00:00Z, 2026-07-31T09:00:15Z)
  activity(vois:applySeal_001, 2026-07-31T09:00:15Z, 2026-07-31T09:00:20Z)
  activity(vois:verifyProof_001, 2026-07-31T10:00:00Z, 2026-07-31T10:00:05Z)
  activity(vois:zeroExposureVerify_001, 2026-07-31T10:05:00Z, 2026-07-31T10:05:03Z)

  /* === Agents === */
  agent(vois:alice, [prov:type='prov:Person', foaf:name="Alice"])
  agent(vois:signingService, [prov:type='prov:SoftwareAgent'])
  agent(vois:auditorBob, [prov:type='prov:Person'])

  /* === Relations: Blueprint Creation === */
  used(vois:createBlueprint_01, vois:blueprintSchema_01, -)
  wasGeneratedBy(vois:blueprint_01, vois:createBlueprint_01, -)
  wasDerivedFrom(vois:blueprint_01, vois:blueprintSchema_01)

  /* === Relations: Claim Assertion === */
  used(vois:assertClaim_001, vois:blueprint_01, -)
  wasGeneratedBy(vois:claim_001, vois:assertClaim_001, -)
  wasAssociatedWith(vois:assertClaim_001, vois:alice, -)
  wasAttributedTo(vois:claim_001, vois:alice)

  /* === Relations: Possession Proof Generation === */
  used(vois:attestProof_001, vois:claim_001, -)
  wasGeneratedBy(vois:possessionProof_001, vois:attestProof_001, -)
  wasAssociatedWith(vois:attestProof_001, vois:alice, -)
  wasAttributedTo(vois:possessionProof_001, vois:alice)
  wasDerivedFrom(vois:possessionProof_001, vois:claim_001)

  /* === Relations: Sealing === */
  used(vois:applySeal_001, vois:possessionProof_001, -)
  used(vois:applySeal_001, vois:claim_001, -)
  wasGeneratedBy(vois:seal_001, vois:applySeal_001, -)
  wasAssociatedWith(vois:applySeal_001, vois:signingService, -)
  wasAttributedTo(vois:seal_001, vois:signingService)
  wasDerivedFrom(vois:seal_001, vois:possessionProof_001)

  /* === Relations: Evidence Package === */
  hadMember(vois:evidencePackage_001, vois:claim_001)
  hadMember(vois:evidencePackage_001, vois:possessionProof_001)
  hadMember(vois:evidencePackage_001, vois:seal_001)

  wasGeneratedBy(vois:evidencePackage_001, vois:attestProof_001, -) ??? needs dedicated assembly activity
  /* Better: create an AssemblyActivity */
  activity(vois:assemble_001, 2026-07-31T09:01:00Z, 2026-07-31T09:01:05Z)
  used(vois:assemble_001, vois:claim_001, -)
  used(vois:assemble_001, vois:possessionProof_001, -)
  used(vois:assemble_001, vois:seal_001, -)
  wasGeneratedBy(vois:evidencePackage_001, vois:assemble_001, -)

  /* === Relations: Classic Verification === */
  used(vois:verifyProof_001, vois:evidencePackage_001, -)
  used(vois:verifyProof_001, vois:seal_001, -)
  wasGeneratedBy(vois:verificationResult_001, vois:verifyProof_001, -)
  wasAssociatedWith(vois:verifyProof_001, vois:auditorBob, -)
  wasAttributedTo(vois:verificationResult_001, vois:auditorBob)

  /* === Relations: Zero-Exposure Verification === */
  /* Commitment is derived from the claim but doesn't reveal it */
  entity(vois:commitment_001, [
    prov:type='vois:CryptographicCommitment',
    vois:commitmentDigest="0xabc..."
  ])
  entity(vois:zeverifyResult_001, [
    prov:type='vois:VerificationResult',
    vois:result='valid'
  ])

  wasDerivedFrom(vois:commitment_001, vois:claim_001)
  used(vois:zeroExposureVerify_001, vois:commitment_001, -)
  /* NOTE: claim_001 is NOT used here — zero exposure */
  wasGeneratedBy(vois:zeverifyResult_001, vois:zeroExposureVerify_001, -)
  wasAssociatedWith(vois:zeroExposureVerify_001, vois:auditorBob, -)
  wasAttributedTo(vois:zeverifyResult_001, vois:auditorBob)

endDocument
```

---

## 6. PROV-O and JSON-LD (PROV-JSONLD)

### 6.1 What is PROV-JSONLD?

PROV-JSONLD (W3C Member Submission, August 2024) is a JSON-LD serialization of PROV designed for:
- **Lightweight**: Natural for web APIs and JSON developers
- **Semantic**: JSON-LD @context maps to PROV-O ontology
- **Efficient**: Object-per-element structure for incremental processing

**Context URL**: `https://openprovenance.org/prov-jsonld/context.jsonld`

**Namespace**: `http://openprovenance.org/prov-jsonld/ns#`

### 6.2 PROV-JSONLD Example

```json
{
  "@context": [
    {
      "xsd": "http://www.w3.org/2001/XMLSchema#",
      "dcterms": "http://purl.org/dc/terms/",
      "vois": "http://vaulted.org/ns/vois#",
      "prov": "http://www.w3.org/ns/prov#",
      "foaf": "http://xmlns.com/foaf/0.1/"
    },
    "https://openprovenance.org/prov-jsonld/context.jsonld"
  ],
  "@graph": [
    {
      "@type": "Entity",
      "@id": "vois:claim_001",
      "vois:predicate": "possesses",
      "vois:subject": "alice",
      "vois:object": "vois:object_42"
    },
    {
      "@type": "Activity",
      "@id": "vois:assertClaim_001",
      "startTime": "2026-07-31T08:30:00Z",
      "endTime": "2026-07-31T08:35:00Z"
    },
    {
      "@type": "Agent",
      "@id": "vois:alice",
      "type": ["prov:Person"],
      "foaf:name": "Alice"
    },
    {
      "@type": "Generation",
      "entity": "vois:claim_001",
      "activity": "vois:assertClaim_001",
      "time": "2026-07-31T08:35:00Z"
    },
    {
      "@type": "Usage",
      "activity": "vois:assertClaim_001",
      "entity": "vois:blueprint_01"
    },
    {
      "@type": "Association",
      "activity": "vois:assertClaim_001",
      "agent": "vois:alice"
    },
    {
      "@type": "Attribution",
      "entity": "vois:claim_001",
      "agent": "vois:alice"
    }
  ]
}
```

### 6.3 PROV-JSONLD Schema for VOIS Types

When extending PROV-JSONLD for VOIS, custom types use the `type` array property:

```json
{
  "@type": "Entity",
  "@id": "vois:blueprint_01",
  "type": ["vois:Blueprint"],
  "vois:version": "1.0.0"
}
```

This maps to RDF:
```turtle
vois:blueprint_01
  a prov:Entity, vois:Blueprint ;
  vois:version "1.0.0" .
```

### 6.4 JSON-LD Context for VOIS

Define a VOIS @context extension:

```json
{
  "@context": {
    "vois": "http://vaulted.org/ns/vois#",
    "Blueprint": "vois:Blueprint",
    "Claim": "vois:Claim",
    "Seal": "vois:Seal",
    "PossessionProof": "vois:PossessionProof",
    "EvidencePackage": "vois:EvidencePackage",
    "VerificationResult": "vois:VerificationResult",
    "CryptographicCommitment": "vois:CryptographicCommitment",
    "sealDigest": {"@id": "vois:sealDigest", "@type": "xsd:hexBinary"},
    "provenanceHash": {"@id": "vois:provenanceHash", "@type": "xsd:hexBinary"},
    "predicate": "vois:predicate",
    "result": "vois:result"
  }
}
```

---

## 7. PROV XML Serialization

### 7.1 Overview

- **Spec**: W3C Working Group Note (30 April 2013)
- **File suffix**: `.provx`
- **Media type**: `application/provenance+xml`
- **Namespace**: `http://www.w3.org/ns/prov#`
- **Schema**: W3C XML Schema (modular design, multiple schema files)

### 7.2 Basic PROV XML Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<prov:document xmlns:prov="http://www.w3.org/ns/prov#"
               xmlns:vois="http://vaulted.org/ns/vois#"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <!-- Namespace declarations -->
  <prov:namespace prefix="vois" uri="http://vaulted.org/ns/vois#"/>

  <!-- Entity -->
  <prov:entity prov:id="vois:claim_001">
    <prov:type>vois:Claim</prov:type>
    <vois:predicate>possesses</vois:predicate>
    <vois:subject>alice</vois:subject>
    <vois:object>vois:object_42</vois:object>
  </prov:entity>

  <!-- Activity -->
  <prov:activity prov:id="vois:assertClaim_001"
                 prov:startTime="2026-07-31T08:30:00Z"
                 prov:endTime="2026-07-31T08:35:00Z"/>

  <!-- Agent -->
  <prov:agent prov:id="vois:alice">
    <prov:type>prov:Person</prov:type>
  </prov:agent>

  <!-- Relations -->
  <prov:wasGeneratedBy>
    <prov:entity prov:ref="vois:claim_001"/>
    <prov:activity prov:ref="vois:assertClaim_001"/>
    <prov:time>2026-07-31T08:35:00Z</prov:time>
  </prov:wasGeneratedBy>

  <prov:used>
    <prov:activity prov:ref="vois:assertClaim_001"/>
    <prov:entity prov:ref="vois:blueprint_01"/>
  </prov:used>

  <prov:wasAssociatedWith>
    <prov:activity prov:ref="vois:assertClaim_001"/>
    <prov:agent prov:ref="vois:alice"/>
  </prov:wasAssociatedWith>

  <prov:wasAttributedTo>
    <prov:entity prov:ref="vois:claim_001"/>
    <prov:agent prov:ref="vois:alice"/>
  </prov:wasAttributedTo>

  <prov:wasDerivedFrom>
    <prov:generatedEntity prov:ref="vois:possessionProof_001"/>
    <prov:usedEntity prov:ref="vois:claim_001"/>
  </prov:wasDerivedFrom>

</prov:document>
```

### 7.3 Qualified Relations in XML

```xml
<prov:wasGeneratedBy>
  <prov:entity prov:ref="vois:claim_001"/>
  <prov:activity prov:ref="vois:assertClaim_001"/>
  <prov:time>2026-07-31T08:35:00Z</prov:time>
  <prov:role>vois:assertedClaim</prov:role>
</prov:wasGeneratedBy>
```

---

## 8. TypeScript Tools/Libraries for PROV Generation

### 8.1 Available Libraries

| Library | Status | Description |
|---|---|---|
| **ProvJS** (`openprov/provjs`) | Legacy / archived | JavaScript implementation of W3C PROV data model. Supports PROV-N and PROV-JSON generation. Not actively maintained. |
| **PROV-JSONLD** (`openprov/prov-jsonld`) | Current (2024) | Reference implementation for the PROV-JSONLD spec. Provides JSON Schema and @context files. |
| **@jsonld-ex/core** | Active | JavaScript/TypeScript implementation of JSON-LD 1.2 extensions with `toProvO` / `fromProvO` functions for PROV-O RDF graph conversion. |
| **@one137th/mesh** | Active | TypeScript RDF semantic graph layer with W3C PROV-O support for audit trails. Includes SPARQL queries, JSON-LD/CRDT bridge. |
| **ProvToolbox** (Java) | Active | Java toolkit with `provconvert` CLI. Converts between PROV-N, PROV-XML, RDF, PROV-JSONLD, and graphical formats. Useful as a backend validator. |

### 8.2 Recommended Approach for VOIS

**Primary strategy**: Build native TypeScript PROV generation using the PROV-JSONLD schema and context, rather than depending on unmaintained libraries.

```typescript
// === TypeScript PROV Builder for VOIS ===

interface ProvenanceGraph {
  "@context": any[];
  "@graph": ProvNode[];
}

type ProvNode = Entity | Activity | Agent | QualifiedRelation;

interface Entity {
  "@type": "Entity" | "Entity"[];
  "@id": string;
  type?: string[];
  location?: string[];
  label?: string[];
  [key: string]: any;
}

interface Activity {
  "@type": "Activity";
  "@id": string;
  startTime?: string;
  endTime?: string;
  [key: string]: any;
}

interface Agent {
  "@type": "Agent";
  "@id": string;
  type?: string[];
  [key: string]: any;
}

interface Generation {
  "@type": "Generation";
  entity: string;
  activity: string;
  time?: string;
  role?: string;
  [key: string]: any;
}

// Builder class
class ProvGraphBuilder {
  private graph: ProvNode[] = [];
  private prefixes: Record<string, string> = {
    prov: "http://www.w3.org/ns/prov#",
    vois: "http://vaulted.org/ns/vois#",
  };

  addEntity(id: string, attrs?: Record<string, any>): this {
    this.graph.push({ "@type": "Entity", "@id": id, ...attrs });
    return this;
  }

  addActivity(id: string, startTime?: string, endTime?: string): this {
    this.graph.push({
      "@type": "Activity",
      "@id": id,
      ...(startTime && { startTime }),
      ...(endTime && { endTime }),
    });
    return this;
  }

  addAgent(id: string, type?: string[]): this {
    this.graph.push({
      "@type": "Agent",
      "@id": id,
      ...(type && { type }),
    });
    return this;
  }

  addGeneration(entity: string, activity: string, time?: string): this {
    this.graph.push({
      "@type": "Generation",
      entity,
      activity,
      ...(time && { time }),
    });
    return this;
  }

  addUsage(activity: string, entity: string): this {
    this.graph.push({ "@type": "Usage", activity, entity });
    return this;
  }

  addAssociation(activity: string, agent: string): this {
    this.graph.push({ "@type": "Association", activity, agent });
    return this;
  }

  addAttribution(entity: string, agent: string): this {
    this.graph.push({ "@type": "Attribution", entity, agent });
    return this;
  }

  addDerivation(generatedEntity: string, usedEntity: string): this {
    this.graph.push({ "@type": "Derivation", generatedEntity, usedEntity });
    return this;
  }

  addMembership(collection: string, item: string): this {
    this.graph.push({ "@type": "Membership", collection, item });
    return this;
  }

  build(): ProvenanceGraph {
    return {
      "@context": [
        this.prefixes,
        "https://openprovenance.org/prov-jsonld/context.jsonld",
      ],
      "@graph": this.graph,
    };
  }

  toJsonLd(): string {
    return JSON.stringify(this.build(), null, 2);
  }
}

// === Usage Example ===
const prov = new ProvGraphBuilder()
  .addEntity("vois:claim_001", {
    type: ["vois:Claim"],
    "vois:predicate": "possesses",
  })
  .addActivity("vois:attest_001", "2026-07-31T09:00:00Z", "2026-07-31T09:00:10Z")
  .addAgent("vois:alice", ["prov:Person"])
  .addGeneration("vois:claim_001", "vois:attest_001")
  .addUsage("vois:attest_001", "vois:blueprint_01")
  .addAssociation("vois:attest_001", "vois:alice")
  .addAttribution("vois:claim_001", "vois:alice")
  .build();
```

### 8.3 Validation Tooling

- **ProvToolbox `provconvert`** (Java): Can validate and convert between all PROV serializations. Run as a subprocess if needed.
- **PROV-JSONLD Schema**: The JSON Schema from the W3C submission (`schema.json`) validates PROV-JSONLD output.
- **SPARQL Validation**: Use `@one137th/mesh` or similar RDF library to query and validate the PROV graph.

---

## 9. Limitations: What VOIS Primitives Don't Map Cleanly to PROV

| VOIS Primitive | Mapping Issue | Workaround |
|---|---|---|
| **Cryptographic Zero-Exposure Proofs** | PROV assumes visibility of `used` entities; zero-exposure intentionally hides the input | Model the commitment as a derived entity; the raw claim is never referenced in the verification provenance trail |
| **Continuous/Streaming Attestation** | PROV models discrete activities with start/end times | Model each attestation window as a separate activity instance, or use `prov:Collection` with `hadMember` for stream windows |
| **Time-Locked / Expiring Evidence** | PROV has `invalidatedAtTime` but no built-in expiry semantics | Use `prov:Invalidation` activity triggered by a timer agent. Add `vois:expiresAt` custom attribute. |
| **Hierarchical Blueprint Composition** | PROV `hadMember` on collections is flat; no native tree/hierarchy | Layer RDF reification with `vois:parentBlueprint` / `vois:childBlueprint` properties |
| **Threshold / Multi-Signature Seals** | `wasAttributedTo` is typically single-agent | Use qualified `prov:Association` with multiple agents and a `vois:threshold` role attribute, or use `vois:MultiSignatureSeal` as a collection of seal entities |
| **Revocation of Prior Attestations** | PROV has no direct "revokes" relation | Model revocation as an activity with `wasGeneratedBy` a `vois:Revocation` entity, connected via `wasDerivedFrom` to the original attestation |
| **Off-Chain / On-Chain Duality** | PROV doesn't distinguish trust domains | Use `prov:Bundle` boundaries to separate on-chain vs off-chain provenance statements. Bundle each domain separately. |
| **Gas Costs / Execution Economics** | Not a PROV concept | Add custom `vois:gasUsed`, `vois:feeAmount`, `vois:executionCost` attributes to activities |
| **Selective Disclosure (Merkle/Sparse)** | PROV assumes uniform entity granularity | Model the disclosed slice as a specialization of the full claim: `specializationOf(disclosedSlice, fullClaim)` |

---

## 10. Summary and Recommendations

### 10.1 What Maps Well

- **Entities → prov:Entity**: Vaulted objects, claims, blueprints, seals, proofs → natural fit
- **Activities → prov:Activity**: Sealing, attestation, verification, assembly → natural fit
- **Agents → prov:Agent**: People, organizations, signing services → natural fit with subtypes
- **Relations → PROV Relations**: Generation, usage, attribution, derivation, association → excellent coverage
- **Temporal ordering → time annotations**: `startedAtTime`, `endedAtTime`, `generatedAtTime`
- **Evidence packaging → prov:Bundle**: Self-contained provenance document

### 10.2 What Needs Custom Extension

- **VOIS domain types**: Create `vois:VaultedObject`, `vois:Blueprint`, `vois:Claim`, `vois:Seal`, `vois:PossessionProof`, `vois:EvidencePackage`, `vois:VerificationResult` as subclasses of `prov:Entity`
- **VOIS activities**: Create `vois:SealingActivity`, `vois:AttestationActivity`, `vois:VerificationActivity` as subclasses of `prov:Activity`
- **Cryptographic attributes**: Add `vois:sealDigest`, `vois:provenanceHash`, `vois:commitmentDigest` as typed properties
- **VOIS context**: Define a custom JSON-LD @context file for VOIS extensions to PROV-O

### 10.3 Recommended Architecture

```
┌─────────────────────────────────────────────────────┐
│                   VOIS Simulator                      │
├─────────────────────────────────────────────────────┤
│  Provenance Graph (internal representation)           │
│  - Typed Node/Edge model                              │
│  - Cross-reference integrity checks                   │
├─────────────────────────────────────────────────────┤
│  PROV Exporter (TypeScript)                           │
│  - PROV-JSONLD export (primary)                       │
│  - PROV-N export (debugging)                          │
│  - PROV-XML export (interop)                          │
├─────────────────────────────────────────────────────┤
│  ProvToolbox (subprocess validation)                  │
│  - Schema validation                                  │
│  - Cross-format conversion                            │
└─────────────────────────────────────────────────────┘
```

### 10.4 Key URIs for VOIS Extension

```
Namespace: http://vaulted.org/ns/vois#
JSON-LD Context: http://vaulted.org/ns/vois/context.jsonld
JSON Schema: http://vaulted.org/ns/vois/schema.json
```

---

## 11. References

1. **PROV-DM**: W3C Recommendation, "PROV-DM: The PROV Data Model" — https://www.w3.org/TR/prov-dm/
2. **PROV-O**: W3C Recommendation, "PROV-O: The PROV Ontology" — https://www.w3.org/TR/prov-o/
3. **PROV-N**: W3C Recommendation, "PROV-N: The Provenance Notation" — https://www.w3.org/TR/prov-n/
4. **PROV-Primer**: W3C Working Group Note, "PROV Model Primer" — https://www.w3.org/TR/prov-primer/
5. **PROV-XML**: W3C Working Group Note, "PROV-XML: The PROV XML Schema" — https://www.w3.org/TR/prov-xml/
6. **PROV-JSONLD**: W3C Member Submission (2024), "The PROV-JSONLD Serialization" — https://www.w3.org/submissions/2024/SUBM-prov-jsonld-20240825/
7. **ProvToolbox**: Java toolkit for PROV — https://lucmoreau.github.io/ProvToolbox/
8. **ProvJS**: JavaScript PROV library — https://github.com/openprov/provjs
9. **@jsonld-ex/core**: NPM JSON-LD extensions with PROV-O support — https://www.npmjs.com/package/@jsonld-ex/core
10. **@one137th/mesh**: TypeScript RDF with PROV-O — https://www.npmjs.com/package/@one137th/mesh
