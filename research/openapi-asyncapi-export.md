# OpenAPI and AsyncAPI Export — Research

## OpenAPI 3.1 Basics

OpenAPI describes RESTful API endpoints:

```yaml
openapi: "3.1.0"
info:
  title: "VOIS Insurance Claims API"
  version: "0.1.0"
paths:
  /claims:
    post:
      summary: "Submit a claim"
      operationId: submitClaim
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ClaimSubmission"
      responses:
        "201":
          description: "Claim created"
```

## AsyncAPI 3.0 Basics

AsyncAPI describes event-driven/channel-based APIs:

```yaml
asyncapi: "3.0.0"
info:
  title: "VOIS Insurance Events"
  version: "0.1.0"
channels:
  claim.submitted:
    address: "claim.submitted"
    messages:
      claimSubmitted:
        $ref: "#/components/messages/ClaimSubmitted"
operations:
  onClaimSubmitted:
    action: receive
    channel:
      $ref: "#/channels/claim.submitted"
```

## VOIS Event → API Operation Mapping

| VOIS Event Type | OpenAPI Operation | HTTP Method | Endpoint Pattern |
|----------------|-------------------|-------------|------------------|
| creation | Create | POST | `/objects/{type}` |
| modification | Update | PATCH | `/objects/{id}` |
| verification | Verify | POST | `/objects/{id}/verify` |
| sealing | Seal | POST | `/objects/{id}/seal` |
| transfer | Transfer | POST | `/objects/{id}/transfer` |
| attestation | Attest | POST | `/objects/{id}/attest` |
| audit | Get audit | GET | `/objects/{id}/audit` |

## VOIS Event → AsyncAPI Channel Mapping

| VOIS Event Type | AsyncAPI Channel | Payload |
|----------------|------------------|---------|
| creation | `object.created` | Object metadata + actor |
| modification | `object.modified` | Diff + actor + authority |
| verification | `object.verified` | Verification result + proof ref |
| sealing | `object.sealed` | Seal metadata + authority |
| transfer | `object.transferred` | From + to + proof ref |
| verification.failed | `verification.failed` | Failure reason + evidence |
| authority.delegated | `authority.delegated` | Delegation chain |
| audit.generated | `audit.generated` | Audit record reference |

## Example: Insurance Claim OpenAPI Spec

```yaml
openapi: "3.1.0"
info:
  title: "VOIS Insurance Claims API"
  description: "API generated from Vaulted Objects Infrastructure blueprint: Insurance Claim Evidence Workflow"
  version: "0.1.0"

servers:
  - url: https://api.vaulted.ventures/v1

paths:
  /claims:
    post:
      summary: "Submit a claim"
      operationId: submitClaim
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ClaimSubmission"
      responses:
        "201":
          description: "Claim created. Returns claim ID for evidence submission."

  /claims/{claimId}/evidence:
    post:
      summary: "Submit evidence with origin seal"
      operationId: submitEvidence
      parameters:
        - name: claimId
          in: path
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              $ref: "#/components/schemas/EvidenceSubmission"
      responses:
        "201":
          description: "Evidence sealed."
          headers:
            X-Proof-Reference:
              schema: { type: string }
              description: "Reference to the generated seal proof"

  /claims/{claimId}/verify:
    post:
      summary: "Verify evidence chain"
      operationId: verifyEvidence
      parameters:
        - name: claimId
          in: path
          required: true
          schema: { type: string }
      responses:
        "200":
          description: "Verification result"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/VerificationResult"

  /claims/{claimId}/seal:
    post:
      summary: "Seal adjudication decision"
      operationId: sealAdjudication
      parameters:
        - name: claimId
          in: path
          required: true
          schema: { type: string }
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/AdjudicationDecision"
      responses:
        "201":
          description: "Decision sealed."
          headers:
            X-Seal-Reference:
              schema: { type: string }

components:
  schemas:
    ClaimSubmission:
      type: object
      properties:
        policyId: { type: string }
        claimantId: { type: string }
        description: { type: string }
        incidentDate: { type: string, format: date }
      required: [policyId, claimantId]

    EvidenceSubmission:
      type: object
      properties:
        file: { type: string, format: binary }
        evidenceType: { type: string, enum: [photo, document, report] }

    VerificationResult:
      type: object
      properties:
        verified: { type: boolean }
        sealCount: { type: integer }
        chainValid: { type: boolean }
        proofReferences: { type: array, items: { type: string } }

    AdjudicationDecision:
      type: object
      properties:
        outcome: { type: string, enum: [approved, denied, pending_review] }
        amount: { type: number }
        notes: { type: string }

  securitySchemes:
    VaultedObjectsAuth:
      type: http
      scheme: bearer
      description: "Zero-Exposure ITS authentication token"
```

## Example: Insurance Claim AsyncAPI Spec

```yaml
asyncapi: "3.0.0"
info:
  title: "VOIS Insurance Events"
  version: "0.1.0"
  description: "Event channels generated from VOIS Insurance Claim Evidence Workflow blueprint"

servers:
  production:
    host: events.vaulted.ventures
    protocol: mqtt
    description: "Production event bus"

channels:
  object.created:
    address: "vois.object.created"
    messages:
      objectCreated:
        $ref: "#/components/messages/ObjectCreated"

  object.sealed:
    address: "vois.object.sealed"
    messages:
      objectSealed:
        $ref: "#/components/messages/ObjectSealed"

  verification.result:
    address: "vois.verification.result"
    messages:
      verificationResult:
        $ref: "#/components/messages/VerificationResult"

  verification.failed:
    address: "vois.verification.failed"
    messages:
      verificationFailed:
        $ref: "#/components/messages/VerificationFailed"

operations:
  onClaimSubmitted:
    action: receive
    channel:
      $ref: "#/channels/object.created"

  onEvidenceSealed:
    action: receive
    channel:
      $ref: "#/channels/object.sealed"

  onVerificationResult:
    action: receive
    channel:
      $ref: "#/channels/verification.result"

components:
  messages:
    ObjectCreated:
      summary: "A new vaulted object was created"
      payload:
        type: object
        properties:
          objectId: { type: string }
          objectType: { type: string }
          actorId: { type: string }
        required: [objectId, objectType]

    ObjectSealed:
      summary: "An object was sealed by an authority"
      payload:
        type: object
        properties:
          objectId: { type: string }
          sealType: { type: string }
          authorityId: { type: string }
          proofRef: { type: string }

    VerificationResult:
      summary: "Evidence chain verification result"
      payload:
        type: object
        properties:
          objectId: { type: string }
          verified: { type: boolean }
          proofCount: { type: integer }

    VerificationFailed:
      summary: "Evidence chain verification failed"
      payload:
        type: object
        properties:
          objectId: { type: string }
          reason: { type: string }
          failedProofRef: { type: string }
```

## Security Scheme: ITS Zero-Exposure Auth

OpenAPI's `securitySchemes` doesn't have a built-in type for ITS Zero-Exposure auth. Options:

1. **Bearer token** (`type: http, scheme: bearer`) — most pragmatic. The bearer token is an
   ITS possession proof encoded as a JWT or similar.
2. **Custom** (`type: http, scheme: vois-its-token`) — custom scheme, needs documentation.
3. **API Key** (`type: apiKey`) — simplest but least expressive.

**Recommendation:** Use `bearer` token with a description noting it's an ITS possession proof.

## Tools & Libraries

| Library | Language | Purpose |
|---------|----------|---------|
| openapi-typescript | TypeScript | Generate TS types from OpenAPI specs |
| @asyncapi/parser | TypeScript | Validate and parse AsyncAPI documents |
| @apiture/openapi-down-convert | TypeScript | Convert between OpenAPI versions |
| swagger-ui | TypeScript | Render OpenAPI specs as interactive docs |
| asyncapi-react | TypeScript | Render AsyncAPI specs as HTML |

## Limitations

1. **No ITS-native auth schemes** — OpenAPI has no built-in security scheme for
   Zero-Exposure/possession-proof authentication.
2. **No event-level proof metadata** — AsyncAPI messages carry proofs as payload fields,
   not as protocol-level attestations.
3. **REST vs event overlap** — Many VOIS events can be both REST endpoints (synchronous)
   and event channels (asynchronous). The blueprint export should include both and let
   the implementation choose.
4. **No state machine semantics** — Neither OpenAPI nor AsyncAPI can express lifecycle
   state transitions (e.g. `claim.submitted → policy_verified → evidence_sealed`).
   These need to be documented in description fields or supplementary state machine
   diagrams.
