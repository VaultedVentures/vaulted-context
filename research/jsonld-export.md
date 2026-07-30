# JSON-LD Semantic Export for VOIS Blueprints

> Research document — Vaulted Objects Infrastructure Simulator (VOIS)
> Date: 2026-07-31

---

## Table of Contents

1. [JSON-LD Basics](#1-json-ld-basics)
2. [Defining a Vaulted Objects Vocabulary / `@context`](#2-defining-a-vaulted-objects-vocabulary)
3. [VOIS Primitive → JSON-LD Node Mapping](#3-vois-primitive--json-ld-node-mapping)
4. [Schema.org Alignment](#4-schemaorg-alignment)
5. [PROV-O Alignment](#5-prov-o-alignment)
6. [Example: Insurance Blueprint in JSON-LD](#6-example-insurance-blueprint-in-json-ld)
7. [Publishing a Stable VOIS Context URL](#7-publishing-a-stable-vois-context-url)
8. [TypeScript Libraries for JSON-LD](#8-typescript-libraries-for-json-ld)

---

## 1. JSON-LD Basics

JSON-LD (JSON for Linked Data) is a W3C Recommendation (1.1, July 2020) that serialises Linked Data in plain JSON. Any valid JSON object is already a valid JSON-LD document — adding `@context` is what gives it semantic meaning.

### Core Keywords

| Keyword | Purpose | Example |
|---------|---------|---------|
| `@context` | Maps short terms to IRIs (vocabulary + definitions) | `"@context": "https://schema.org/"` |
| `@type` | Declares the RDF type of a node | `"@type": "vois:InfrastructureFunction"` |
| `@id` | Unique IRI identifier for a node | `"@id": "urn:vois:function:identity-binding"` |
| `@graph` | Collection of named nodes (top-level array) | `"@graph": [{...}, {...}]` |
| `@vocab` | Default vocabulary prefix (inside `@context`) | `"@vocab": "https://vaulted.ventures/vois/context#/"` |

### Processing Modes

- **Compaction**: Apply `@context` to shorten a fully expanded document → most readable output
- **Expansion**: Remove `@context` to get fully expanded IRIs → for programmatic consumption
- **Flattening**: Extract all named nodes into `@graph` with `@id` references
- **Framing**: Shape a JSON-LD document by a template → ideal for API responses

### Design Principles for VOIS

1. Every **Vaulted Object**, **Function**, **Connection**, and **Simulation** gets a stable `@id`
2. The `@context` is a single stable URL (e.g., `https://vaulted.ventures/vois/context.jsonld`)
3. Types follow the ontology: `vois:InfrastructureFunction`, `vois:VaultedObject`, `vois:ProvenanceRecord`, etc.
4. Properties use camelCase aliases for readability (e.g., `setupCost` → `vois:setupCost`)
5. References between objects use `@id` (not embedded objects)

---

## 2. Defining a Vaulted Objects Vocabulary

A **`@context` document** is published JSON that defines how short terms expand to full IRIs. It acts as the schema/vocabulary bridge.

### Namespace Strategy

```
Prefix:  vois
Base:    https://vaulted.ventures/vois/context#
```

Example context skeleton:

```json
{
  "@context": {
    "@version": 1.1,
    "@vocab": "https://vaulted.ventures/vois/context#",

    "schema": "http://schema.org/",
    "prov":   "http://www.w3.org/ns/prov#",
    "xsd":    "http://www.w3.org/2001/XMLSchema#",
    "rdfs":   "http://www.w3.org/2000/01/rdf-schema#",

    "id": "@id",
    "type": "@type",
    "graph": "@graph",

    "InfrastructureFunction": { "@id": "vois:InfrastructureFunction" },
    "VaultedObject":          { "@id": "vois:VaultedObject" },
    "ProvenanceRecord":      { "@id": "vois:ProvenanceRecord" },
    "ChainOfCustody":        { "@id": "vois:ChainOfCustody" },
    "SourceVerification":    { "@id": "vois:SourceAttestation" },
    "DigitalSeal":           { "@id": "vois:DigitalSeal" },
    "DigitalSignature":      { "@id": "vois:DigitalSignature" },
    "EvidencePackage":       { "@id": "vois:EvidencePackage" },
    "SecureMessage":         { "@id": "vois:SecureMessage" },
    "AccessPolicy":          { "@id": "vois:AccessPolicy" },
    "PaymentInstruction":    { "@id": "vois:PaymentInstruction" },
    "Settlement":            { "@id": "vois:Settlement" },
    "IdentityBinding":       { "@id": "vois:IdentityBinding" },
    "DeviceBinding":         { "@id": "vois:DeviceBinding" },
    "OrgIdentity":           { "@id": "vois:OrgIdentity" },
    "BusinessProfile":       { "@id": "vois:BusinessProfile" },
    "WorkflowTemplate":      { "@id": "vois:WorkflowTemplate" },
    "Simulation":            { "@id": "vois:Simulation" },

    "name":         { "@id": "schema:name" },
    "description":  { "@id": "schema:description" },

    "inputs":  { "@id": "vois:hasInput",  "@container": "@set" },
    "outputs": { "@id": "vois:hasOutput", "@container": "@set" },
    "connections": { "@id": "vois:hasConnection", "@container": "@set" },
    "nodes":    { "@id": "vois:hasNode", "@container": "@set" },
    "scenarios": { "@id": "vois:hasScenario", "@container": "@set" },

    "setupCost":           { "@id": "vois:setupCost",           "@type": "xsd:decimal" },
    "recurringCostPerMonth": { "@id": "vois:recurringCostPerMonth", "@type": "xsd:decimal" },
    "transactionFeePercent": { "@id": "vois:transactionFeePercent", "@type": "xsd:decimal" },
    "expectedRevenueUpliftPercent":  { "@id": "vois:expectedRevenueUpliftPercent", "@type": "xsd:decimal" },
    "expectedCostReductionPercent":  { "@id": "vois:expectedCostReductionPercent", "@type": "xsd:decimal" },
    "expectedFraudReductionPercent": { "@id": "vois:expectedFraudReductionPercent", "@type": "xsd:decimal" },
    "expectedComplianceSavingPercent": { "@id": "vois:expectedComplianceSavingPercent", "@type": "xsd:decimal" },

    "sourceNodeId": { "@id": "vois:sourceNode" },
    "targetNodeId": { "@id": "vois:targetNode" },
    "sourcePortId": { "@id": "vois:sourcePort" },
    "targetPortId": { "@id": "vois:targetPort" },
    "dataType":     { "@id": "vois:dataType" },
    "category":     { "@id": "vois:category" },
    "technicalDescription": { "@id": "vois:technicalDescription" },

    "annualRevenue":          { "@id": "vois:annualRevenue", "@type": "xsd:decimal" },
    "annualCostBase":         { "@id": "vois:annualCostBase", "@type": "xsd:decimal" },
    "annualTransactionCount": { "@id": "vois:annualTransactionCount", "@type": "xsd:integer" },
    "averageTransactionValue": { "@id": "vois:averageTransactionValue", "@type": "xsd:decimal" },
    "currentFraudRatePercent": { "@id": "vois:currentFraudRatePercent", "@type": "xsd:decimal" },
    "annualComplianceCost":   { "@id": "vois:annualComplianceCost", "@type": "xsd:decimal" },
    "estimatedDataRiskExposure": { "@id": "vois:estimatedDataRiskExposure", "@type": "xsd:decimal" },

    "adoptionCurve":   { "@id": "vois:adoptionCurve" },
    "curveType":       { "@id": "vois:curveType" },
    "startYear":       { "@id": "vois:startYear", "@type": "xsd:integer" },
    "inflectionYear":  { "@id": "vois:inflectionYear", "@type": "xsd:integer" },
    "maxAdoption":     { "@id": "vois:maxAdoption", "@type": "xsd:decimal" },
    "steepness":       { "@id": "vois:steepness", "@type": "xsd:decimal" },
    "discountRatePercent": { "@id": "vois:discountRatePercent", "@type": "xsd:decimal" },
    "years": { "@id": "vois:projectionYears", "@type": "xsd:integer" },

    "createdAt": { "@id": "vois:createdAt", "@type": "xsd:dateTime" },
    "updatedAt": { "@id": "vois:updatedAt", "@type": "xsd:dateTime" },

    "identifier": { "@id": "schema:identifier" }
  }
}
```

### Key Design Decisions

- **`xsd:` typed properties** — numeric fields are explicitly typed so SPARQL queries can do arithmetic
- **`@container: @set`** — arrays are declared as unordered sets (the default and simplest)
- **`schema:` re-use** — `name`, `description`, and `identifier` map to Schema.org for basic interoperability
- **`prov:` prefix** left available for future provenance annotation (see §5)
- **`@vocab` set to `vois:` namespace** — any unrecognised term falls through to the vaulted context

---

## 3. VOIS Primitive → JSON-LD Node Mapping

### 3.1 InfrastructureFunction

```json
{
  "@id": "urn:vois:function:identity-binding",
  "@type": "vois:InfrastructureFunction",
  "name": "Identity Binding",
  "description": "Bind a cryptographic identity to a person or entity with ITS security.",
  "category": "Identity",
  "technicalDescription": "Creates a vaulted object that binds a cryptographic public key to verified identity attributes, anchored by ITS digital signatures.",
  "inputs": [{
    "@id": "urn:vois:port:identity-binding:identity-in",
    "label": "Identity Claim",
    "dataType": "identity"
  }],
  "outputs": [{
    "@id": "urn:vois:port:identity-binding:identity-out",
    "label": "Bound Identity",
    "dataType": "identity"
  }],
  "setupCost": 50000,
  "recurringCostPerMonth": 2000,
  "expectedFraudReductionPercent": 15,
  "expectedRevenueUpliftPercent": 2
}
```

### 3.2 SimulatorNodeData (placed node on canvas)

```json
{
  "@id": "urn:vois:node:sim-abc123:identity-binding",
  "@type": "vois:SimulatorNode",
  "name": "Identity Binding",
  "functionId": { "@id": "urn:vois:function:identity-binding" },
  "position": { "x": 50, "y": 100 },
  "config": {}
}
```

### 3.3 SimulatorConnection

```json
{
  "@id": "urn:vois:conn:conn-1",
  "@type": "vois:Connection",
  "sourceNodeId": { "@id": "urn:vois:node:sim-abc123:identity-binding" },
  "sourcePortId": "identity-out",
  "targetNodeId": { "@id": "urn:vois:node:sim-abc123:vaulted-object-create" },
  "targetPortId": "vo-data-in",
  "dataType": "identity"
}
```

### 3.4 CommercialAssumptions / Scenario

```json
{
  "@id": "urn:vois:scenario:scen-base",
  "@type": "vois:Scenario",
  "name": "Base case",
  "setupCost": 50000,
  "recurringCostPerMonth": 2000,
  "expectedRevenueUpliftPercent": 3,
  "discountRatePercent": 12,
  "projectionYears": 5,
  "adoptionCurve": {
    "@type": "vois:AdoptionCurve",
    "curveType": "sCurve",
    "startYear": 2026,
    "inflectionYear": 2028,
    "maxAdoption": 0.7,
    "steepness": 0.8
  }
}
```

### 3.5 BusinessProfile

```json
{
  "@id": "urn:vois:profile:bp-abc123",
  "@type": "vois:BusinessProfile",
  "annualRevenue": 10000000,
  "annualCostBase": 7000000,
  "annualTransactionCount": 100000,
  "averageTransactionValue": 100,
  "currentFraudRatePercent": 2,
  "annualComplianceCost": 500000,
  "estimatedDataRiskExposure": 5000000
}
```

### 3.6 Full Simulation (top-level)

```json
{
  "@context": "https://vaulted.ventures/vois/context.jsonld",
  "@graph": [
    {
      "@id": "urn:vois:sim:abc123",
      "@type": "vois:Simulation",
      "name": "Secure Digital Asset Licensing",
      "industry": "Digital Media",
      "createdAt": "2026-07-31T10:00:00Z",
      "updatedAt": "2026-07-31T10:00:00Z",
      "businessProfile": { "@id": "urn:vois:profile:bp-abc123" },
      "nodes": [
        { "@id": "urn:vois:node:sim-abc123:identity-binding" },
        { "@id": "urn:vois:node:sim-abc123:vaulted-object-create" }
      ],
      "connections": [
        { "@id": "urn:vois:conn:conn-1" }
      ],
      "scenarios": [
        { "@id": "urn:vois:scenario:scen-base" },
        { "@id": "urn:vois:scenario:scen-conservative" }
      ]
    }
  ]
}
```

### 3.7 Category Mapping

| VOIS Primitive (TypeScript) | JSON-LD `@type` | Description |
|----------------------------|-----------------|-------------|
| `InfrastructureFunction` | `vois:InfrastructureFunction` | Blueprint function definition |
| `SimulatorNodeData` | `vois:SimulatorNode` | Placed node instance on canvas |
| `SimulatorConnection` | `vois:Connection` | Edge/patch cord between ports |
| `CommercialAssumptions` | `vois:CommercialAssumptions` | Embedded in Scenario |
| `AdoptionCurve` | `vois:AdoptionCurve` | S-curve parameters |
| `BusinessProfile` | `vois:BusinessProfile` | Business context |
| `Scenario` | `vois:Scenario` | Scenario with assumptions |
| `Simulation` | `vois:Simulation` | Top-level document |
| `WorkflowTemplate` | `vois:WorkflowTemplate` | Saved blueprint template |
| `WebsiteCopyBlock` | `vois:ContentBlock` | Copy/content block |

---

## 4. Schema.org Alignment

Schema.org provides well-known, search-engine-friendly types that overlap with VOIS concepts. Selective alignment increases discoverability.

### Recommended Mappings

| VOIS Term | Schema.org Type / Property | Rationale |
|-----------|---------------------------|-----------|
| `Simulation` | `schema:Dataset` | A simulation is a collection of data |
| `BusinessProfile` | `schema:Organization` + `schema:BusinessEntityType` | Describes a business |
| `name` | `schema:name` | Universal label |
| `description` | `schema:description` | Universal summary |
| `identifier` | `schema:identifier` | Stable ID |
| `InfrastructureFunction` | `schema:SoftwareApplication` | A function is a capability/component |
| `EvidencePackage` | `schema:DigitalDocument` + `schema:CreativeWork` | A document artifact |
| `createdAt` | `schema:dateCreated` | Creation timestamp |
| `updatedAt` | `schema:dateModified` | Modification timestamp |
| `category` | `schema:applicationCategory` | Software category |
| `recurringCostPerMonth` | `schema:offer` → `schema:PriceSpecification` | Cost representation |

### Example: Simulation with Schema.org overlay

```json
{
  "@context": [
    "https://vaulted.ventures/vois/context.jsonld",
    { "schema": "http://schema.org/" }
  ],
  "@graph": [
    {
      "@id": "urn:vois:sim:abc123",
      "@type": ["vois:Simulation", "schema:Dataset"],
      "schema:name": "Secure Digital Asset Licensing",
      "schema:description": "Model how creators license digital assets with secure identity, provenance, micro-payments and revenue splitting.",
      "schema:dateCreated": "2026-07-31T10:00:00Z",
      "schema:dateModified": "2026-07-31T10:00:00Z",
      "name": "Secure Digital Asset Licensing",
      "industry": "Digital Media"
    }
  ]
}
```

### Alignment by Use Case

| Use Case | Schema.org Type | Why |
|----------|----------------|-----|
| Blueprint as a web page | `schema:TechArticle` | For search engine indexing of published blueprints |
| Licensing workflow | `schema:Product` + `schema:Offer` | For commerce-oriented blueprints |
| Carbon provenance | `schema:CreativeWork` + `schema:Claim` | For evidence-based workflows |
| Agentic software provenance | `schema:SoftwareSourceCode` | For code provenance |

**Policy**: Add Schema.org types via `@type` array (e.g., `["vois:Simulation", "schema:Dataset"]`) but keep VOIS properties in the `vois:` namespace. This is a "lite" alignment — full RDFS/OWL equivalence is not required.

---

## 5. PROV-O Alignment

PROV-O (http://www.w3.org/ns/prov#) is the W3C provenance ontology. Core classes:

| PROV-O Class | Meaning | VOIS Equivalent |
|-------------|---------|-----------------|
| `prov:Entity` | A physical, digital, conceptual, or other thing | `VaultedObject`, `ProvenanceRecord`, `EvidencePackage` |
| `prov:Activity` | Something that occurs over a period | `InfrastructureFunction` execution, node processing |
| `prov:Agent` | Something that bears some form of responsibility | `IdentityBinding`, `OrgIdentity`, signing party |
| `prov:Collection` | A entity that aggregates others | `EvidencePackage`, `Simulation` |

### Key PROV-O Properties for VOIS

| Property | Domain → Range | VOIS Mapping |
|----------|---------------|--------------|
| `prov:wasGeneratedBy` | Entity → Activity | VaultedObject → InfrastructureFunction invocation |
| `prov:used` | Activity → Entity | InfrastructureFunction → its Input VaultedObjects |
| `prov:wasDerivedFrom` | Entity → Entity | ProvenanceRecord → prior ProvenanceRecord |
| `prov:wasAttributedTo` | Entity → Agent | VaultedObject → IdentityBinding (signer) |
| `prov:wasAssociatedWith` | Activity → Agent | Function → identity that authorised it |
| `prov:actedOnBehalfOf` | Agent → Agent | Delegation chain |
| `prov:hadMember` | Collection → Entity | EvidencePackage → contained VaultedObjects |
| `prov:atTime` | Activity → xsd:dateTime | Timestamp of function execution |
| `prov:generated` | Activity → Entity | Inverse of wasGeneratedBy |
| `prov:invalidatedAtTime` | Entity → xsd:dateTime | When a sealed object expires |

### Blueprint → PROV-O Example

A **Provenance Record** in VOIS maps naturally to the PROV-O data model:

```json
{
  "@id": "urn:vois:provenance:pr-001",
  "@type": ["vois:ProvenanceRecord", "prov:Entity"],
  "prov:wasDerivedFrom": { "@id": "urn:vois:provenance:pr-000" },
  "prov:wasGeneratedBy": {
    "@id": "urn:vois:activity:record-provenance",
    "@type": ["vois:ProvenanceRecording", "prov:Activity"],
    "prov:used": { "@id": "urn:vois:vaulted-object:vo-001" },
    "prov:wasAssociatedWith": { "@id": "urn:vois:function:identity-binding" },
    "prov:atTime": "2026-07-31T10:00:00Z"
  },
  "prov:wasAttributedTo": { "@id": "urn:vois:identity:org-vaulted" }
}
```

### PROV-O Integration Plan

1. **Export** — Every `SimulatorNodeData` node that processes data becomes a `prov:Activity` in the JSON-LD export
2. **Chains** — `ChainOfCustody` records link via `prov:wasDerivedFrom`
3. **Agents** — Identity-bound entities (`IdentityBinding`, `OrgIdentity`) become `prov:Agent` nodes
4. **Attestation** — ITS digital seals and signatures are `prov:Entity` with `prov:wasGeneratedBy` pointing to the sealing activity

### Full PROV-O Namespace

```json
{
  "@context": {
    "prov": "http://www.w3.org/ns/prov#",
    "Entity": "prov:Entity",
    "Activity": "prov:Activity",
    "Agent": "prov:Agent",
    "wasGeneratedBy": "prov:wasGeneratedBy",
    "used": "prov:used",
    "wasDerivedFrom": "prov:wasDerivedFrom",
    "wasAttributedTo": "prov:wasAttributedTo",
    "wasAssociatedWith": "prov:wasAssociatedWith",
    "actedOnBehalfOf": "prov:actedOnBehalfOf",
    "hadMember": "prov:hadMember",
    "atTime": "prov:atTime"
  }
}
```

---

## 6. Example: Insurance Blueprint in JSON-LD

This example converts a simplified **insurance claims processing** blueprint (noting that VOIS currently has 5 templates: Secure Digital Asset Licensing, Sovereign Data Trust, Secure Procurement, Carbon Provenance, Agentic Software Provenance).

We use the existing **Secure Procurement Workflow** as a close insurance analogue (counterparty verification → signature → evidence → payment) and render it as a complete JSON-LD export.

```json
{
  "@context": "https://vaulted.ventures/vois/context.jsonld",
  "@graph": [
    {
      "@id": "urn:vois:sim:insurance-claims",
      "@type": "vois:Simulation",
      "name": "Insurance Claims Processing",
      "industry": "Insurance",
      "description": "Model how insurance claims are verified, attested, and paid using secure counterparty verification, ITS signatures, and evidence packages.",
      "createdAt": "2026-07-31T10:00:00Z",
      "updatedAt": "2026-07-31T10:00:00Z",
      "businessProfile": { "@id": "urn:vois:profile:ins-co" },
      "nodes": [
        { "@id": "urn:vois:node:ins-1:counterparty-verification" },
        { "@id": "urn:vois:node:ins-1:role-authority" },
        { "@id": "urn:vois:node:ins-1:its-digital-signature" },
        { "@id": "urn:vois:node:ins-1:evidence-package" },
        { "@id": "urn:vois:node:ins-1:payment-initiation" }
      ],
      "connections": [
        { "@id": "urn:vois:conn:ins-1:cv-to-ra" },
        { "@id": "urn:vois:conn:ins-1:ra-to-sig" },
        { "@id": "urn:vois:conn:ins-1:sig-to-ep" },
        { "@id": "urn:vois:conn:ins-1:ep-to-pi" }
      ],
      "scenarios": [
        { "@id": "urn:vois:scenario:ins-base" },
        { "@id": "urn:vois:scenario:ins-upside" }
      ]
    },

    {
      "@id": "urn:vois:profile:ins-co",
      "@type": "vois:BusinessProfile",
      "annualRevenue": 50000000,
      "annualCostBase": 35000000,
      "annualTransactionCount": 500000,
      "averageTransactionValue": 500,
      "currentFraudRatePercent": 5,
      "annualComplianceCost": 2000000,
      "estimatedDataRiskExposure": 25000000
    },

    {
      "@id": "urn:vois:node:ins-1:counterparty-verification",
      "@type": "vois:SimulatorNode",
      "name": "Counterparty Verification",
      "functionId": { "@id": "urn:vois:function:counterparty-verification" },
      "position": { "x": 50, "y": 150 }
    },
    {
      "@id": "urn:vois:node:ins-1:role-authority",
      "@type": "vois:SimulatorNode",
      "name": "Role & Authority Verification",
      "functionId": { "@id": "urn:vois:function:role-authority" },
      "position": { "x": 350, "y": 50 }
    },
    {
      "@id": "urn:vois:node:ins-1:its-digital-signature",
      "@type": "vois:SimulatorNode",
      "name": "ITS Digital Signature",
      "functionId": { "@id": "urn:vois:function:its-digital-signature" },
      "position": { "x": 350, "y": 250 }
    },
    {
      "@id": "urn:vois:node:ins-1:evidence-package",
      "@type": "vois:SimulatorNode",
      "name": "Evidence Package Generation",
      "functionId": { "@id": "urn:vois:function:evidence-package" },
      "position": { "x": 650, "y": 150 }
    },
    {
      "@id": "urn:vois:node:ins-1:payment-initiation",
      "@type": "vois:SimulatorNode",
      "name": "Payment Initiation",
      "functionId": { "@id": "urn:vois:function:payment-initiation" },
      "position": { "x": 950, "y": 150 }
    },

    {
      "@id": "urn:vois:conn:ins-1:cv-to-ra",
      "@type": "vois:Connection",
      "sourceNodeId": { "@id": "urn:vois:node:ins-1:counterparty-verification" },
      "sourcePortId": "cp-verification-out",
      "targetNodeId": { "@id": "urn:vois:node:ins-1:role-authority" },
      "targetPortId": "role-identity-in",
      "dataType": "attestation"
    },
    {
      "@id": "urn:vois:conn:ins-1:ra-to-sig",
      "@type": "vois:Connection",
      "sourceNodeId": { "@id": "urn:vois:node:ins-1:role-authority" },
      "sourcePortId": "role-verification-out",
      "targetNodeId": { "@id": "urn:vois:node:ins-1:its-digital-signature" },
      "targetPortId": "sig-identity-in",
      "dataType": "attestation"
    },
    {
      "@id": "urn:vois:conn:ins-1:sig-to-ep",
      "@type": "vois:Connection",
      "sourceNodeId": { "@id": "urn:vois:node:ins-1:its-digital-signature" },
      "sourcePortId": "sig-signed-out",
      "targetNodeId": { "@id": "urn:vois:node:ins-1:evidence-package" },
      "targetPortId": "ep-sealed-in",
      "dataType": "sealed"
    },
    {
      "@id": "urn:vois:conn:ins-1:ep-to-pi",
      "@type": "vois:Connection",
      "sourceNodeId": { "@id": "urn:vois:node:ins-1:evidence-package" },
      "sourcePortId": "ep-package-out",
      "targetNodeId": { "@id": "urn:vois:node:ins-1:payment-initiation" },
      "targetPortId": "pi-auth-in",
      "dataType": "evidence"
    },

    {
      "@id": "urn:vois:scenario:ins-base",
      "@type": "vois:Scenario",
      "name": "Base case",
      "setupCost": 20000,
      "recurringCostPerMonth": 800,
      "expectedFraudReductionPercent": 20,
      "expectedComplianceSavingPercent": 10,
      "discountRatePercent": 12,
      "projectionYears": 5,
      "adoptionCurve": {
        "@type": "vois:AdoptionCurve",
        "curveType": "sCurve",
        "startYear": 2026,
        "inflectionYear": 2028,
        "maxAdoption": 0.7,
        "steepness": 0.8
      }
    },
    {
      "@id": "urn:vois:scenario:ins-upside",
      "@type": "vois:Scenario",
      "name": "Upside",
      "setupCost": 20000,
      "recurringCostPerMonth": 800,
      "expectedFraudReductionPercent": 35,
      "expectedComplianceSavingPercent": 18,
      "discountRatePercent": 10,
      "projectionYears": 7,
      "adoptionCurve": {
        "@type": "vois:AdoptionCurve",
        "curveType": "sCurve",
        "startYear": 2026,
        "inflectionYear": 2027,
        "maxAdoption": 0.9,
        "steepness": 1.0
      }
    }
  ]
}
```

### With PROV-O annotation (excerpt)

```json
{
  "@context": [
    "https://vaulted.ventures/vois/context.jsonld",
    { "prov": "http://www.w3.org/ns/prov#" }
  ],
  "@graph": [
    {
      "@id": "urn:vois:node:ins-1:evidence-package",
      "@type": ["vois:SimulatorNode", "prov:Activity"],
      "prov:used": [
        { "@id": "urn:vois:vaulted-object:sealed-claim-001" },
        { "@id": "urn:vois:vaulted-object:claim-context" }
      ],
      "prov:wasAssociatedWith": { "@id": "urn:vois:function:evidence-package" },
      "prov:generated": { "@id": "urn:vois:vaulted-object:evidence-pkg-001" }
    },
    {
      "@id": "urn:vois:vaulted-object:evidence-pkg-001",
      "@type": ["vois:EvidencePackage", "prov:Entity", "schema:DigitalDocument"],
      "prov:wasGeneratedBy": { "@id": "urn:vois:node:ins-1:evidence-package" },
      "schema:name": "Claim Evidence Package #001"
    }
  ]
}
```

---

## 7. Publishing a Stable VOIS Context URL

### Recommended URL Structure

```
Base URL:   https://vaulted.ventures/vois/
Context:    https://vaulted.ventures/vois/context.jsonld
Ontology:   https://vaulted.ventures/vois/ontology.ttl    (Turtle — for OWL/RDFS tools)
            https://vaulted.ventures/vois/ontology.jsonld  (same content as JSON-LD)
Docs:       https://vaulted.ventures/vois/docs/
Namespace:  https://vaulted.ventures/vois/context#         (fragment-based — every term resolves)
```

### Publishing via Cloudflare Workers (natural fit for vaulted.ventures)

The Vaulted Ventures domain already uses Cloudflare. Serve the `@context` JSON file via a Workers route:

```typescript
// workers/vois-context
export default {
  async fetch(req: Request): Promise<Response> {
    const url = new URL(req.url);

    if (url.pathname === '/vois/context.jsonld') {
      const context = await import('./context.json');
      return new Response(JSON.stringify(context.default), {
        headers: {
          'Content-Type': 'application/ld+json',
          'Access-Control-Allow-Origin': '*',
          'Cache-Control': 'public, max-age=86400, stale-while-revalidate=604800',
          'Vary': 'Accept',
        },
      });
    }

    return new Response('Not Found', { status: 404 });
  },
};
```

### Cool URIs — Best Practices (from W3C)

| Principle | Implementation |
|-----------|---------------|
| **Don't change URIs** | Version the context only when breaking changes are needed |
| **Content negotiation** | Serve `application/ld+json` by default; HTML for human browsing via `Accept: text/html` |
| **Use 301 redirects for old versions** | `/vois/v1/context.jsonld` → permanent redirect to current |
| **Cache aggressively** | `Cache-Control: public, max-age=86400` — the W3C JSON-LD BP doc recommends caching `@context` documents liberally |
| **CORS open** | `Access-Control-Allow-Origin: *` — contexts are fetched cross-origin by JSON-LD processors |
| **Monitor traffic** | Use Cloudflare analytics; the context file may be fetched thousands of times per day |

### Versioning Strategy

```
https://vaulted.ventures/vois/context.jsonld           ← always latest
https://vaulted.ventures/vois/v1/context.jsonld         ← v1 frozen
https://vaulted.ventures/vois/v2/context.jsonld         ← v2 frozen (future)
```

**Semantic versioning in the context document**: embed a `@version` number in the JSON-LD standard `@context`:

```json
{
  "@context": {
    "@version": 1.1,
    "vois": "https://vaulted.ventures/vois/context#",
    "vois-context-version": { "@value": "1.0.0" }
  }
}
```

### Security Considerations

- The `@context` document is **critical security infrastructure** — if compromised, an attacker can remap VOIS terms to arbitrary IRIs
- **Subresource Integrity (SRI)**: publish a hash and serve it alongside the document
- **HTTPS only**: always serve over TLS
- **Cloudflare Workers** provides automatic HTTPS + DDoS protection

---

## 8. TypeScript Libraries for JSON-LD

### Primary: `jsonld` + `@types/jsonld`

The official W3C JSON-LD processor for JavaScript.

```bash
npm install jsonld @types/jsonld
```

**Capabilities:**

| Function | Purpose | Use in VOIS |
|----------|---------|-------------|
| `jsonld.compact(doc, context)` | Apply context to shorten IRIs | Serialise internal graph to user-facing JSON-LD |
| `jsonld.expand(doc)` | Strip context, produce full IRIs | Validate inbound RDF data |
| `jsonld.flatten(doc)` | Extract all @id nodes into @graph | Normalise a simulation for export |
| `jsonld.frame(doc, frame)` | Shape output to match a JSON template | API responses fitting a schema |
| `jsonld.canonize(doc)` | Deterministic N-Quads via RDFC-1.0 | Content-addressed integrity proofs for vaulted objects |
| `jsonld.toRDF(doc)` | Convert to RDF quads | SPARQL querying / triple store ingestion |
| `jsonld.fromRDF(quads)` | Convert RDF quads back to JSON-LD | Read provenance data from triple store |

**Example — exporting a simulation from VOIS:**

```typescript
import * as jsonld from 'jsonld';
import type { Simulation } from './lib/types';

const CONTEXT_URL = 'https://vaulted.ventures/vois/context.jsonld';

async function exportSimulationAsJsonLd(sim: Simulation): Promise<object> {
  // Build the @graph document from simulation data
  const doc = {
    '@context': CONTEXT_URL,
    '@graph': [
      simulationNode(sim),
      ...sim.nodes.map(nodeToJsonLdNode),
      ...sim.connections.map(connToJsonLdConn),
      ...sim.scenarios.map(scenarioToJsonLd),
      businessProfileNode(sim.businessProfile),
    ],
  };

  // Compact to ensure clean output (handles any expansion)
  return await jsonld.compact(doc, CONTEXT_URL);
}
```

### Supporting Libraries

| Library | Package | Purpose |
|---------|---------|---------|
| **schema-dts** | `npm install schema-dts` | TypeScript definitions for Schema.org types |
| **@avensio/jsonld-schema** | `npm install @avensio/jsonld-schema` | Generated TypeScript toolkit for Schema.org JSON-LD |
| **schema-org-adapter** | `npm install schema-org-adapter` | Runtime adapter for Schema.org vocabulary loading |
| **n3.js** | `npm install n3.js` | RDF/SPARQL processing — parse Turtle, write N-Triples |
| **rdf-ext** | `npm install rdf-ext` | Low-level RDF/JS data model (quads, terms, datasets) |
| **solid-client** | `npm install @inrupt/solid-client` | Solid Pod interaction and RDF/JS utilities |

### Recommended Stack

For the VOIS exporter, the minimal stack is:

```bash
npm install jsonld @types/jsonld
# Optional for Schema.org type safety:
npm install schema-dts
# Optional for RDF/Solid integration:
npm install n3.js @rdfjs/dataset
```

### Example TypeScript: `exportToJsonLd.ts`

```typescript
import * as jsonld from 'jsonld';
import type {
  Simulation, SimulatorNodeData, SimulatorConnection, Scenario
} from './lib/types';

const CONTEXT_URL = 'https://vaulted.ventures/vois/context.jsonld';

interface JsonLdNode {
  '@id': string;
  '@type': string;
  [key: string]: unknown;
}

function simulationNode(sim: Simulation): JsonLdNode {
  return {
    '@id': `urn:vois:sim:${sim.id}`,
    '@type': 'vois:Simulation',
    name: sim.name,
    industry: sim.industry,
    createdAt: sim.createdAt,
    updatedAt: sim.updatedAt,
    businessProfile: { '@id': `urn:vois:profile:${sim.id}` },
    nodes: sim.nodes.map(n => ({ '@id': `urn:vois:node:${sim.id}:${n.functionId}` })),
    connections: sim.connections.map(c => ({ '@id': `urn:vois:conn:${c.id}` })),
    scenarios: sim.scenarios.map(s => ({ '@id': `urn:vois:scenario:${s.id}` })),
  };
}

function nodeToJsonLdNode(node: SimulatorNodeData): JsonLdNode {
  return {
    '@id': `urn:vois:node:${node.id}`,
    '@type': 'vois:SimulatorNode',
    name: node.label,
    functionId: { '@id': `urn:vois:function:${node.functionId}` },
    category: node.category,
    'position:x': node.position.x,
    'position:y': node.position.y,
  };
}

function connectionToJsonLd(conn: SimulatorConnection): JsonLdNode {
  return {
    '@id': `urn:vois:conn:${conn.id}`,
    '@type': 'vois:Connection',
    sourceNodeId: { '@id': `urn:vois:node:${conn.sourceNodeId}` },
    sourcePortId: conn.sourcePortId,
    targetNodeId: { '@id': `urn:vois:node:${conn.targetNodeId}` },
    targetPortId: conn.targetPortId,
    dataType: conn.dataType,
  };
}

function scenarioToJsonLd(scenario: Scenario): JsonLdNode {
  return {
    '@id': `urn:vois:scenario:${scenario.id}`,
    '@type': 'vois:Scenario',
    name: scenario.name,
    setupCost: scenario.assumptions.setupCost,
    recurringCostPerMonth: scenario.assumptions.recurringCostPerMonth,
    expectedRevenueUpliftPercent: scenario.assumptions.expectedRevenueUpliftPercent,
    discountRatePercent: scenario.discountRatePercent,
    projectionYears: scenario.years,
    adoptionCurve: scenario.adoptionCurve,
  };
}

async function exportSimulation(sim: Simulation): Promise<object> {
  const doc = {
    '@context': CONTEXT_URL,
    '@graph': [
      simulationNode(sim),
      businessProfileNode(sim),
      ...sim.nodes.map(nodeToJsonLdNode),
      ...sim.connections.map(connectionToJsonLd),
      ...sim.scenarios.map(scenarioToJsonLd),
    ].filter(Boolean),
  };
  return jsonld.compact(doc, CONTEXT_URL);
}

function businessProfileNode(sim: Simulation): JsonLdNode {
  return {
    '@id': `urn:vois:profile:${sim.id}`,
    '@type': 'vois:BusinessProfile',
    annualRevenue: sim.businessProfile.annualRevenue,
    annualCostBase: sim.businessProfile.annualCostBase,
    annualTransactionCount: sim.businessProfile.annualTransactionCount,
    averageTransactionValue: sim.businessProfile.averageTransactionValue,
    currentFraudRatePercent: sim.businessProfile.currentFraudRatePercent,
    annualComplianceCost: sim.businessProfile.annualComplianceCost,
    estimatedDataRiskExposure: sim.businessProfile.estimatedDataRiskExposure,
  };
}

export { exportSimulation, CONTEXT_URL };
```

---

## Appendix A: Implementation Roadmap

| Phase | Task | Effort |
|-------|------|--------|
| **1** | Publish `@context` JSON document at stable URL | 1 day |
| **2** | Create `lib/exportToJsonLd.ts` in VOIS codebase | 1 day |
| **3** | Add "Export as JSON-LD" button to Blueprint view | 0.5 day |
| **4** | Add PROV-O annotation to provenance chain exports | 1 day |
| **5** | Add Schema.org dual-typing for search-engine alignment | 0.5 day |
| **6** | Write integration tests: round-trip JSON-LD → import → expand → compact | 1 day |
| **7** | Publish ontology docs at `/vois/docs/` | 1 day |

## Appendix B: URI Scheme Design

```
urn:vois:sim:{simulationId}
urn:vois:node:{simulationId}:{functionId}
urn:vois:conn:{connectionId}
urn:vois:function:{functionId}         ← canonical (from functions.ts)
urn:vois:profile:{simulationId}
urn:vois:scenario:{scenarioId}
urn:vois:port:{functionId}:{portId}
urn:vois:vaulted-object:{objectId}
urn:vois:provenance:{recordId}
urn:vois:identity:{entityId}
urn:vois:activity:{functionId}:{timestamp}
```

Using `urn:vois:` URNs instead of HTTPS URIs for individual entities:
- Avoids requiring deployed infrastructure for every entity
- Makes export documents self-contained
- Can be upgraded to HTTPS URIs in a later revision
- URNs are valid JSON-LD IRIs per RFC 8141

For the `@context` document itself and the ontology, use HTTPS URIs so JSON-LD processors can fetch them.

## Appendix C: Related Standards & References

| Standard | URL | Relevance |
|----------|-----|-----------|
| JSON-LD 1.1 | https://www.w3.org/TR/json-ld11/ | Core serialization format |
| JSON-LD 1.1 API | https://www.w3.org/TR/json-ld11-api/ | Processing algorithms |
| PROV-O | https://www.w3.org/TR/prov-o/ | Provenance ontology |
| PROV-DM | https://www.w3.org/TR/prov-dm/ | Provenance data model |
| Schema.org | https://schema.org/ | Web vocabulary alignment |
| Cool URIs | https://www.w3.org/TR/cooluris/ | URI design principles |
| Linked Data Best Practices | https://www.w3.org/TR/ld-bp/ | Publishing guidelines |
| JSON-LD Best Practices | https://w3c.github.io/json-ld-bp/ | Context caching, API design |
| RDF 1.1 Concepts | https://www.w3.org/TR/rdf11-concepts/ | Underlying data model |
| OWL 2 RL | https://www.w3.org/TR/owl2-profiles/ | OWL profile used by PROV-O |
