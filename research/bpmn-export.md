# BPMN 2.0 Export Mapping for VOIS Blueprints

> **Status:** Research Complete  
> **Date:** 2026-07-31  
> **Schema Version:** `vois-blueprint 0.1.0`  
> **BPMN Version:** 2.0 (OMG Specification)

---

## 1. BPMN 2.0 XML Structure Basics

BPMN 2.0 XML is governed by the OMG BPMN 2.0 XSD
(`BPMN20.xsd` at `http://www.omg.org/spec/BPMN/20100501/BPMN20.xsd`).
Every valid BPMN document has this canonical structure:

### 1.1 The `<definitions>` Root

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions
  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
  xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC"
  xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI"
  xmlns:vois="http://vaultedobjects.io/schema/vois/1.0"
  targetNamespace="http://www.omg.org/spec/BPMN/20100524/MODEL"
  id="definitions-claim-workflow">
```

### 1.2 Core Elements

| BPMN Element | XML Tag | Purpose |
|---|---|---|
| **Process** | `<process>` | Container for flow elements (tasks, gateways, events, sequence flows). Has `id`, `name`, `isExecutable`. |
| **Start Event** | `<startEvent>` | Where a process begins. Mandatory. Can have timing/message/trigger details. |
| **End Event** | `<endEvent>` | Where a process terminates. |
| **Task** | `<task>` | Generic atomic activity. Abstract — use typed subtypes. |
| **User Task** | `<userTask>` | Performed by a human actor. Can have `candidateGroups`, `assignee`. |
| **Service Task** | `<serviceTask>` | Performed by a service/automation. Maps to agents and automated events. |
| **Send Task** | `<sendTask>` | Sends a message to an external participant. |
| **Receive Task** | `<receiveTask>` | Waits for a message from an external participant. |
| **Exclusive Gateway** | `<exclusiveGateway>` | XOR decision — exactly one outgoing path. Has `gatewayDirection` attribute. |
| **Parallel Gateway** | `<parallelGateway>` | AND fork/join — all outgoing paths execute. |
| **Inclusive Gateway** | `<inclusiveGateway>` | OR — one or more outgoing paths. |
| **Event-Based Gateway** | `<eventBasedGateway>` | Reacts to the first occurring event. |
| **Sequence Flow** | `<sequenceFlow>` | Directed edge connecting flow nodes. Has `sourceRef`, `targetRef`, optional `<conditionExpression>`. |
| **Pool** | `<participant>` (in `<collaboration>`) | Represents a participant/organisation. References a process via `processRef`. |
| **Lane** | `<lane>` (in `<laneSet>`) | Sub-division of a pool. Groups flow nodes by role/actor. Uses `<flowNodeRef>`. |
| **Data Object** | `<dataObject>` | Information flowing through the process (documents, records). |
| **Data Object Reference** | `<dataObjectReference>` | A state-specific occurrence of a data object. Refers to a `<dataObject>` via `dataObjectRef`. |
| **Data Store** | `<dataStore>` | Persistent storage accessible across the process. |
| **Data Store Reference** | `<dataStoreReference>` | A specific use of a data store within a process. |
| **Message Flow** | `<messageFlow>` | Communication between two pools (across boundaries). |
| **Association** | `<association>` | Connects artefacts (text annotations, data objects) to flow nodes. |
| **Text Annotation** | `<textAnnotation>` | Free-text note attached to the diagram. |
| **Documentation** | `<documentation>` | Inline documentation within any BPMN element. |
| **Extension Elements** | `<extensionElements>` | Container for custom extensions (vendor or domain-specific). |

### 1.3 Collaboration (Multi-Pool) Structure

For multi-party workflows (the norm in VOIS), you wrap everything in a `<collaboration>`:

```xml
<definitions ...>
  <collaboration id="collaboration-claim">
    <participant id="pool-policyholder" name="Policyholder" processRef="process-policyholder" />
    <participant id="pool-insurer" name="Insurer" processRef="process-insurer" />
    <messageFlow id="mf-001" name="Claim Submitted"
      sourceRef="start-claim" targetRef="task-receive-claim" />
  </collaboration>

  <!-- Per-participant processes -->
  <process id="process-policyholder" name="Policyholder Process" isExecutable="false">
    ...
  </process>
  <process id="process-insurer" name="Insurer Process" isExecutable="true">
    <laneSet id="laneSet-insurer">
      <lane id="lane-adjuster" name="Claims Adjuster">
        <flowNodeRef>task-review</flowNodeRef>
      </lane>
      ...
    </laneSet>
    ...
  </process>
</definitions>
```

### 1.4 Condition Expressions on Sequence Flows

For exclusive gateways, outgoing sequence flows carry `<conditionExpression>`:

```xml
<sequenceFlow id="sf-approved" name="Yes" sourceRef="gateway-approve" targetRef="task-process">
  <conditionExpression xsi:type="tFormalExpression">${approved == true}</conditionExpression>
</sequenceFlow>
<sequenceFlow id="sf-rejected" name="No" sourceRef="gateway-approve" targetRef="task-reject">
  <conditionExpression xsi:type="tFormalExpression">${approved == false}</conditionExpression>
</sequenceFlow>
```

### 1.5 BPMNDI Diagram Layout

The BPMN Diagram Interchange (BPMNDI) layer, while optional for semantics, is essential for rendering:

```xml
<bpmndi:BPMNDiagram id="BPMNDiagram_1">
  <bpmndi:BPMNPlane id="BPMNPlane_1" bpmnElement="collaboration-claim">
    <bpmndi:BPMNShape id="pool-policyholder_di" bpmnElement="pool-policyholder" isHorizontal="true">
      <omgdc:Bounds x="120" y="60" width="1200" height="200" />
    </bpmndi:BPMNShape>
    <bpmndi:BPMNShape id="start-claim_di" bpmnElement="start-claim">
      <omgdc:Bounds x="180" y="130" width="36" height="36" />
    </bpmndi:BPMNShape>
    <bpmndi:BPMNEdge id="sf-001_di" bpmnElement="sf-001">
      <omgdi:waypoint x="216" y="148" />
      <omgdi:waypoint x="300" y="148" />
    </bpmndi:BPMNEdge>
  </bpmndi:BPMNPlane>
</bpmndi:BPMNDiagram>
```

---

## 2. Mapping Table: VOIS Concepts → BPMN Elements

### 2.1 Top-Level Blueprint

| VOIS Field | BPMN Mapping | Notes |
|---|---|---|
| `format` (const: "vois-blueprint") | → `exporter` attribute on `<definitions>` | `exporter="VOIS Simulator" exporterVersion="0.1.0"` |
| `version` | → stored in `vois:version` extension on `<definitions>` | BPMN has no native version field |
| `name` | → `name` attribute on `<collaboration>` | Also populate process `name` attributes |
| `industry` / `scenario` | → stored as `vois:industry`, `vois:scenario` extensions | No native BPMN equivalent |
| `description` | → `<documentation>` on `<collaboration>` | |

### 2.2 Actors → Pools & Lanes

| VOIS | BPMN | Strategy |
|---|---|---|
| `actors[]` with distinct `trust_boundary` | → `<participant>` (Pool) per trust boundary | Each trust boundary is one pool |
| `actors[]` within same trust boundary | → `<lane>` inside the pool's `<laneSet>` | Role-based lanes within the organisation pool |
| `actors[].type` | → stored as `vois:actorType` extension | Enum: human, organisation, system, role |

**Mapping Algorithm:**

1. Group actors by `trust_boundary` → each group becomes a pool (`<participant>`).
2. Within each pool, each actor becomes a lane (`<lane>`).
3. Process is per-pool. Internal flow uses `<sequenceFlow>`.
4. Cross-pool communication uses `<messageFlow>`.

### 2.3 Agents → Service Tasks or Lanes

| VOIS | BPMN | Strategy |
|---|---|---|
| `agents[]` | → `<serviceTask>` within the delegating actor's lane | Autonomous agents are automated services |
| `agents[].delegated_by` | → `vois:delegatedBy` extension on the `<serviceTask>` | Links agent back to delegating authority |
| `agents[].capabilities` | → `vois:capabilities` extension array | |
| `agents[].constraints` | → `<documentation>` | |

If an agent has its own trust boundary, it gets its own lane rather than being a task.

### 2.4 Digital Objects → Data Objects / Data Stores

| VOIS | BPMN | Strategy |
|---|---|---|
| `objects[]` | → `<dataObject>` + `<dataObjectReference>` | Each object gets a definition at process level |
| `objects[].is_vaulted=true` | → `vois:vaulted="true"` extension on `<dataObject>` | VOIS-specific |
| `objects[].possessed_by` | → `vois:possessedBy` extension | Maps to entity ref |
| `objects[].lifecycle_state` | → `<dataState>` within `<dataObjectReference>` | BPMN data state |
| `objects[].lifecycle_transitions` | → stored as TBD; could use state machine extension | |

VOIS objects that are acted upon by events become attached via `<dataInputAssociation>` / `<dataOutputAssociation>` on the relevant task or event.

### 2.5 Authorities → BPMN Extensions or Resources

| VOIS | BPMN | Strategy |
|---|---|---|
| `authorities[]` | → `<resource>` or `vois:authority` extension | BPMN has no native "authority" concept |
| `authorities[].seal_types` | → `vois:sealTypes` extension array | |
| `authorities[].verification_method` | → `vois:verificationMethod` extension | |
| `authorities[].trust_anchor` | → `vois:trustAnchor` extension | |

**Recommendation:** Map authorities to `<resourceRole>` on tasks/gateways they govern, or use a dedicated `vois:authority` extension element.

### 2.6 Events → BPMN Activities & Events

| VOIS Event Type | BPMN Element | Notes |
|---|---|---|
| `creation` | → `<startEvent>` or `<task>` | If it initiates the workflow, use startEvent. If mid-stream, use a task with `vois:eventType="creation"`. |
| `modification` | → `<userTask>` or `<scriptTask>` | With `vois:eventType="modification"` |
| `transfer` | → Sequence flow + message flow combination | Object passes between actors |
| `verification` | → `<businessRuleTask>` or `<serviceTask>` | Automated check |
| `sealing` | → `<serviceTask>` with `vois:eventType="sealing"` | ITS seal generation |
| `attestation` | → `<userTask>` with `vois:eventType="attestation"` | Human attestation |
| `possession_change` | → `vois:possessionChange` extension on sequence flow | |
| `authority_delegation` | → Not directly mapped; stored as extension | |
| `audit` | → `<serviceTask>` or boundary event on process | |
| `destruction` | → `<endEvent>` | Termination lifecycle |

**Event structure:**

```xml
<serviceTask id="evt-003" name="Incident Photo Attested">
  <extensionElements>
    <vois:eventType value="sealing" />
    <vois:actorRef ref="act-001" />
    <vois:objectRef ref="obj-003" />
    <vois:authorityRef ref="auth-001" />
    <vois:evidenceRefs>
      <vois:evidenceRef ref="prf-002" />
    </vois:evidenceRefs>
  </extensionElements>
  <incoming>...</incoming>
  <outgoing>...</outgoing>
</serviceTask>
```

### 2.7 Controls → Gateways

| VOIS Control Type | BPMN Element | Notes |
|---|---|---|
| `approval_gate` | → `<exclusiveGateway>` | Decision point (approve/reject) |
| `verification_check` | → `<exclusiveGateway>` or `<businessRuleTask>` + gateway | Can be split into the check (task) + decision (gateway) |
| `policy_enforcement` | → `<businessRuleTask>` with condition | Automated rule enforcement |
| `audit_trigger` | → Boundary event or `<intermediateCatchEvent>` | |
| `escalation` | → `<boundaryEvent>` with escalation trigger | Attached to the task that may time out |
| `human_review` | → `<userTask>` | Manual review step |
| `seal_requirement` | → Gateway condition checking `vois:vaulted` status | |
| `possession_check` | → Gateway with `vois:possessionCheck` extension | |

**Control pattern (approval gate):**

```xml
<exclusiveGateway id="ctl-001" name="Policy Valid?" gatewayDirection="Diverging">
  <extensionElements>
    <vois:control>
      <vois:controlType>verification_check</vois:controlType>
      <vois:controlledBy ref="auth-001" />
      <vois:governanceRules>
        <vois:rule>Claim must match active policy</vois:rule>
      </vois:governanceRules>
    </vois:control>
  </extensionElements>
  <incoming>...</incoming>
  <outgoing>...</outgoing>
</exclusiveGateway>
```

### 2.8 Proofs → Documentation & Extensions

| VOIS Proof Type | BPMN Mapping | Notes |
|---|---|---|
| `digital_seal` | → `<documentation>` + `vois:proof` extension on producing task | |
| `its_signature` | → `vois:proof` extension with `type="its_signature"` | |
| `possession_proof` | → `vois:possessionProof` extension on data object reference | |
| `verification_record` | → `vois:proof` extension on verification task | |
| `audit_log` | → `<documentation>` on process level | |
| `attestation` | → `vois:proof` extension on attestation task | |
| `certificate` | → `vois:proof` extension | |

Each proof with `consumed_by` → the consuming event's input association references it.

### 2.9 Nodes & Edges → BPMN Flow Elements + BPMNDI

| VOIS | BPMN |
|---|---|
| `nodes[].type=actor/agent` | → pool/lane assignment (BPMNShape bounds) |
| `nodes[].type=event` | → startEvent, endEvent, task, businessRuleTask, serviceTask |
| `nodes[].type=control` | → exclusiveGateway, parallelGateway |
| `nodes[].type=object` | → dataObjectReference |
| `nodes[].type=authority` | → not rendered as flow node; attached as extension |
| `nodes[].position` | → `<omgdc:Bounds>` in BPMNDI |
| `edges[].type=flow` | → `<sequenceFlow>` |
| `edges[].type=data_flow` | → `<dataAssociation>` (`dataInputAssociation` / `dataOutputAssociation`) |
| `edges[].type=control` | → `<sequenceFlow>` from gateway to next element |
| `edges[].type=possession` | → `vois:edgeType="possession"` extension on association |
| `edges[].type=authority` | → `vois:edgeType="authority"` on association |
| `edges[].type=verification` | → `vois:edgeType="verification"` |
| `edges[].type=message` | → `<messageFlow>` (cross-pool) |
| `edges[].type=delegation` | → `vois:edgeType="delegation"` |

---

## 3. VOIS-Specific Extensions (The Extension Elements Namespace)

BPMN 2.0 has a native extensibility mechanism via `<extensionElements>` that appears on
**any** flow element. You define a custom namespace (e.g. `vois:`) and nest extension
data inside it.

### 3.1 Defining the VOIS Namespace

```xml
<definitions
  xmlns:vois="http://vaultedobjects.io/schema/vois/1.0"
  ...>
```

### 3.2 Extension Definitions (VOIS XSD Skeleton)

You should create a formal XSD at a well-known URL:
`https://raw.githubusercontent.com/VaultedVentures/vaulted-context/main/schemas/vois-bpmn.xsd`

The extension namespace should cover:

| Extension Element | Parent | Purpose |
|---|---|---|
| `vois:eventType` | task/serviceTask/userTask | Captures VOIS event type enum (creation, sealing, etc.) |
| `vois:actorRef` | task, gateway | Links BPMN element to VOIS actor entity |
| `vois:objectRef` | task, gateway | Links BPMN element to VOIS digital object |
| `vois:authorityRef` | task, gateway | Links BPMN element to VOIS authority |
| `vois:evidenceRefs` | task | Set of proof references produced/consumed |
| `vois:proof` | task | Full proof metadata (type, name, produced_by, consumed_by) |
| `vois:control` | gateway | Full control metadata (type, governance rules, controlled_by) |
| `vois:possession` | dataObjectReference, sequenceFlow | Possession tracking (current possessor, change events) |
| `vois:seal` | task, dataObject | Seal metadata (seal type, authority, timestamp) |
| `vois:vaulted` | dataObject | Whether the object is ITS-bound |
| `vois:capabilities` | serviceTask | Agent capabilities |
| `vois:delegatedBy` | serviceTask | Delegation chain |
| `vois:industry` | definitions | Industry sector |
| `vois:scenario` | definitions | Workflow scenario |
| `vois:blueprintVersion` | definitions | VOIS schema version |
| `vois:risk` | definitions, task | Risk metadata (severity, mitigation) |
| `vois:edgeType` | sequenceFlow, messageFlow | Semantic edge type |

### 3.3 Complete Extension Example

```xml
<serviceTask id="evt-005" name="Evidence Chain Verified">
  <extensionElements>
    <vois:eventType value="verification" />
    <vois:actorRef ref="agt-002" entityType="agent" />
    <vois:objectRef ref="obj-004" entityType="object" />
    <vois:authorityRef ref="auth-002" entityType="authority" />
    <vois:evidenceRefs>
      <vois:evidenceRef ref="prf-001" />
      <vois:evidenceRef ref="prf-002" />
      <vois:evidenceRef ref="prf-003" />
    </vois:evidenceRefs>
    <vois:proof>
      <vois:proofId>prf-004</vois:proofId>
      <vois:proofType>verification_record</vois:proofType>
      <vois:proofName>Evidence Chain Verification</vois:proofName>
    </vois:proof>
  </extensionElements>
  <incoming>...</incoming>
  <outgoing>...</outgoing>
</serviceTask>
```

---

## 4. Example: Insurance Claim Evidence → BPMN XML

Below is the BPMN 2.0 export of the `insurance-claim-evidence` blueprint.

### Strategy

1. **Pools by trust boundary:**
   - Pool 1: "External — Claimant" (Policyholder)
   - Pool 2: "External — Accredited" (Independent Assessor)
   - Pool 3: "External — Supervisory" (Regulator)
   - Pool 4: "Internal — Insurer" (Claims Adjuster, Claims Intake Agent, Evidence Verifier)
2. **Lanes** for each actor within their pool.
3. **Events → tasks** with `vois:eventType` extensions.
4. **Controls → exclusive gateways** with `vois:control` extensions.
5. **Objects → dataObjectReferences** at process level.
6. **Cross-pool communication → messageFlows**.
7. **Authorities → shared resources** referenced via `vois:authorityRef`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions
  xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:bpmndi="http://www.omg.org/spec/BPMN/20100524/DI"
  xmlns:omgdc="http://www.omg.org/spec/DD/20100524/DC"
  xmlns:omgdi="http://www.omg.org/spec/DD/20100524/DI"
  xmlns:vois="http://vaultedobjects.io/schema/vois/1.0"
  targetNamespace="http://www.omg.org/spec/BPMN/20100524/MODEL"
  id="definitions-insurance-claim"
  exporter="VOIS Simulator"
  exporterVersion="0.1.0">

  <!-- ===== DEFINITIONS-LEVEL META ===== -->
  <documentation>
    <![CDATA[
      VOIS Blueprint: Insurance Claim Evidence Workflow
      Industry: insurance
      Scenario: Claim evidence submission, verification, sealing, and adjudication
    ]]>
  </documentation>

  <extension>
    <vois:blueprintVersion>0.1.0</vois:blueprintVersion>
    <vois:industry>insurance</vois:industry>
    <vois:scenario>Claim evidence submission, verification, sealing, and adjudication</vois:scenario>
  </extension>

  <!-- ===== AUTHORITIES (Shared Resources) ===== -->
  <resource id="auth-001" name="Insurer Authority">
    <extensionElements>
      <vois:authority>
        <vois:authorityType>certifying_authority</vois:authorityType>
        <vois:sealTypes>
          <vois:sealType>origin_seal</vois:sealType>
          <vois:sealType>authority_seal</vois:sealType>
          <vois:sealType>attestation</vois:sealType>
        </vois:sealTypes>
        <vois:verificationMethod>Possession proof + seal chain</vois:verificationMethod>
      </vois:authority>
    </extensionElements>
  </resource>
  <resource id="auth-002" name="Accredited Assessor Authority">
    <extensionElements>
      <vois:authority>
        <vois:authorityType>trusted_third_party</vois:authorityType>
        <vois:sealTypes>
          <vois:sealType>attestation</vois:sealType>
          <vois:sealType>integrity_seal</vois:sealType>
        </vois:sealTypes>
        <vois:verificationMethod>Multi-party verification</vois:verificationMethod>
      </vois:authority>
    </extensionElements>
  </resource>
  <resource id="auth-003" name="Industry Regulator">
    <extensionElements>
      <vois:authority>
        <vois:authorityType>regulatory_body</vois:authorityType>
        <vois:sealTypes>
          <vois:sealType>authority_seal</vois:sealType>
        </vois:sealTypes>
        <vois:verificationMethod>Hierarchical trust chain</vois:verificationMethod>
      </vois:authority>
    </extensionElements>
  </resource>

  <!-- ===== MESSAGES ===== -->
  <message id="msg-001" name="Claim Submission" />
  <message id="msg-002" name="Incident Photo" />
  <message id="msg-003" name="Assessor Report" />
  <message id="msg-004" name="Adjudication Decision" />
  <message id="msg-005" name="Audit Notification" />

  <!-- ===== COLLABORATION (Pools) ===== -->
  <collaboration id="collaboration-claim" name="Insurance Claim Evidence Workflow">
    <participant id="pool-claimant" name="Policyholder (Claimant)" processRef="process-claimant" />
    <participant id="pool-insurer" name="Insurer" processRef="process-insurer" />
    <participant id="pool-assessor" name="Independent Assessor" processRef="process-assessor" />
    <participant id="pool-regulator" name="Regulator" processRef="process-regulator" />

    <!-- Cross-pool message flows -->
    <messageFlow id="mf-claim-submitted" name="Claim Submission"
      sourceRef="evt-001-claim-submitted" targetRef="task-receive-claim" messageRef="msg-001" />
    <messageFlow id="mf-photo-submitted" name="Incident Photo"
      sourceRef="evt-003-photo-attested" targetRef="task-receive-photo" messageRef="msg-002" />
    <messageFlow id="mf-report-sealed" name="Assessor Report"
      sourceRef="evt-004-report-sealed" targetRef="task-receive-report" messageRef="msg-003" />
    <messageFlow id="mf-adjudication" name="Adjudication Decision"
      sourceRef="evt-006-adjudication" targetRef="task-receive-decision" messageRef="msg-004" />
    <messageFlow id="mf-audit" name="Audit Notification"
      sourceRef="task-audit" targetRef="evt-007-audit" messageRef="msg-005" />
  </collaboration>

  <!-- ================================================================== -->
  <!-- PROCESS: Policyholder (Pool: claimant)                             -->
  <!-- ================================================================== -->
  <process id="process-claimant" name="Policyholder Process" isExecutable="false">
    <laneSet id="laneSet-claimant">
      <lane id="lane-policyholder" name="Policyholder">
        <flowNodeRef>evt-001-claim-submitted</flowNodeRef>
        <flowNodeRef>evt-003-photo-attested</flowNodeRef>
      </lane>
    </laneSet>

    <startEvent id="start-claim" name="Policyholder initiates claim">
      <outgoing>sf-internal-001</outgoing>
    </startEvent>

    <sequenceFlow id="sf-internal-001" sourceRef="start-claim" targetRef="evt-001-claim-submitted" />

    <serviceTask id="evt-001-claim-submitted" name="Claim Submitted">
      <extensionElements>
        <vois:eventType value="creation" />
        <vois:objectRef ref="obj-001" entityType="object" />
      </extensionElements>
      <incoming>sf-internal-001</incoming>
      <outgoing>sf-internal-002</outgoing>
    </serviceTask>

    <sequenceFlow id="sf-internal-002" sourceRef="evt-001-claim-submitted" targetRef="evt-003-photo-attested" />

    <serviceTask id="evt-003-photo-attested" name="Incident Photo Attested">
      <extensionElements>
        <vois:eventType value="sealing" />
        <vois:objectRef ref="obj-003" entityType="object" />
        <vois:authorityRef ref="auth-001" entityType="authority" />
        <vois:evidenceRefs>
          <vois:evidenceRef ref="prf-002" />
        </vois:evidenceRefs>
      </extensionElements>
      <incoming>sf-internal-002</incoming>
      <outgoing>sf-end-001</outgoing>
    </serviceTask>

    <endEvent id="sf-end-001" name="Photos submitted">
      <incoming>sf-end-001</incoming>
    </endEvent>
  </process>

  <!-- ================================================================== -->
  <!-- PROCESS: Insurer (Pool: insurer) — the main process                 -->
  <!-- ================================================================== -->
  <process id="process-insurer" name="Insurer Process" isExecutable="true">
    <laneSet id="laneSet-insurer">
      <lane id="lane-intake-agent" name="Claims Intake Agent (AI)">
        <flowNodeRef>task-receive-claim</flowNodeRef>
        <flowNodeRef>evt-002-policy-check</flowNodeRef>
      </lane>
      <lane id="lane-adjuster" name="Claims Adjuster">
        <flowNodeRef>evt-006-adjudication</flowNodeRef>
      </lane>
      <lane id="lane-evidence-verifier" name="Evidence Verifier (AI)">
        <flowNodeRef>task-receive-photo</flowNodeRef>
        <flowNodeRef>task-receive-report</flowNodeRef>
        <flowNodeRef>evt-005-evidence-verified</flowNodeRef>
        <flowNodeRef>ctl-004-verification-gate</flowNodeRef>
      </lane>
    </laneSet>

    <!-- Data Objects -->
    <dataObject id="obj-001-def" name="Claim Submission">
      <extensionElements>
        <vois:objectType>document</vois:objectType>
        <vois:vaulted>false</vois:vaulted>
      </extensionElements>
    </dataObject>
    <dataObject id="obj-002-def" name="Policy Document">
      <extensionElements>
        <vois:objectType>vaulted_object</vois:objectType>
        <vois:vaulted>true</vois:vaulted>
        <vois:lifecycleState>Active</vois:lifecycleState>
      </extensionElements>
    </dataObject>
    <dataObject id="obj-003-def" name="Incident Photo">
      <extensionElements>
        <vois:objectType>evidence</vois:objectType>
        <vois:vaulted>true</vois:vaulted>
        <vois:lifecycleState>Sealed</vois:lifecycleState>
      </extensionElements>
    </dataObject>
    <dataObject id="obj-004-def" name="Assessor Report">
      <extensionElements>
        <vois:objectType>vaulted_object</vois:objectType>
        <vois:vaulted>true</vois:vaulted>
        <vois:lifecycleState>Sealed</vois:lifecycleState>
      </extensionElements>
    </dataObject>
    <dataObject id="obj-005-def" name="Adjudication Decision">
      <extensionElements>
        <vois:objectType>vaulted_object</vois:objectType>
        <vois:vaulted>true</vois:vaulted>
        <vois:lifecycleState>Finalised</vois:lifecycleState>
      </extensionElements>
    </dataObject>
    <dataObject id="obj-006-def" name="Audit Record">
      <extensionElements>
        <vois:objectType>record</vois:objectType>
        <vois:vaulted>true</vois:vaulted>
        <vois:lifecycleState>Active</vois:lifecycleState>
      </extensionElements>
    </dataObject>

    <!-- ===== START ===== -->
    <startEvent id="start-insurer" name="Claim arrives">
      <outgoing>sf-to-receive-claim</outgoing>
    </startEvent>

    <sequenceFlow id="sf-to-receive-claim" sourceRef="start-insurer" targetRef="task-receive-claim" />

    <!-- evt-001 (received on insurer side) -->
    <userTask id="task-receive-claim" name="Receive Claim Submission">
      <extensionElements>
        <vois:eventType value="creation" />
        <vois:objectRef ref="obj-001" entityType="object" />
      </extensionElements>
      <incoming>sf-to-receive-claim</incoming>
      <outgoing>sf-to-policy-check</outgoing>
    </userTask>

    <sequenceFlow id="sf-to-policy-check" sourceRef="task-receive-claim" targetRef="evt-002-policy-check" />

    <!-- evt-002: Policy Check (verification) -->
    <serviceTask id="evt-002-policy-check" name="Policy Check">
      <extensionElements>
        <vois:eventType value="verification" />
        <vois:objectRef ref="obj-002" entityType="object" />
        <vois:authorityRef ref="auth-001" entityType="authority" />
        <vois:evidenceRefs>
          <vois:evidenceRef ref="prf-001" />
        </vois:evidenceRefs>
      </extensionElements>
      <incoming>sf-to-policy-check</incoming>
      <outgoing>sf-to-policy-gate</outgoing>
    </serviceTask>

    <sequenceFlow id="sf-to-policy-gate" sourceRef="evt-002-policy-check" targetRef="ctl-001-policy-gate" />

    <!-- ctl-001: Policy Valid Before Processing (verification_check) -->
    <exclusiveGateway id="ctl-001-policy-gate" name="Policy Valid?" gatewayDirection="Diverging">
      <extensionElements>
        <vois:control>
          <vois:controlType>verification_check</vois:controlType>
          <vois:controlledBy ref="auth-001" />
          <vois:governanceRules>
            <vois:rule>Claim must match active policy</vois:rule>
          </vois:governanceRules>
        </vois:control>
      </extensionElements>
      <incoming>sf-to-policy-gate</incoming>
      <outgoing>sf-policy-valid</outgoing>
      <outgoing>sf-policy-invalid</outgoing>
    </exclusiveGateway>

    <sequenceFlow id="sf-policy-valid" name="Valid" sourceRef="ctl-001-policy-gate" targetRef="task-receive-photo">
      <conditionExpression xsi:type="tFormalExpression">${policyValid == true}</conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="sf-policy-invalid" name="Invalid" sourceRef="ctl-001-policy-gate" targetRef="end-rejected" />

    <endEvent id="end-rejected" name="Claim Rejected">
      <incoming>sf-policy-invalid</incoming>
    </endEvent>

    <!-- Receive photo (from message flow mf-photo-submitted) -->
    <receiveTask id="task-receive-photo" name="Receive Incident Photo" messageRef="msg-002">
      <extensionElements>
        <vois:objectRef ref="obj-003" entityType="object" />
      </extensionElements>
      <incoming>sf-policy-valid</incoming>
      <outgoing>sf-to-receive-report</outgoing>
    </receiveTask>

    <sequenceFlow id="sf-to-receive-report" sourceRef="task-receive-photo" targetRef="task-receive-report" />

    <!-- Receive assessor report (from message flow mf-report-sealed) -->
    <receiveTask id="task-receive-report" name="Receive Assessor Report" messageRef="msg-003">
      <extensionElements>
        <vois:objectRef ref="obj-004" entityType="object" />
      </extensionElements>
      <incoming>sf-to-receive-report</incoming>
      <outgoing>sf-to-evidence-verify</outgoing>
    </receiveTask>

    <sequenceFlow id="sf-to-evidence-verify" sourceRef="task-receive-report" targetRef="evt-005-evidence-verified" />

    <!-- evt-005: Evidence Chain Verified (verification) -->
    <serviceTask id="evt-005-evidence-verified" name="Evidence Chain Verified">
      <extensionElements>
        <vois:eventType value="verification" />
        <vois:objectRef ref="obj-004" entityType="object" />
        <vois:authorityRef ref="auth-002" entityType="authority" />
        <vois:evidenceRefs>
          <vois:evidenceRef ref="prf-001" />
          <vois:evidenceRef ref="prf-002" />
          <vois:evidenceRef ref="prf-003" />
        </vois:evidenceRefs>
        <vois:proof>
          <vois:proofId>prf-004</vois:proofId>
          <vois:proofType>verification_record</vois:proofType>
          <vois:proofName>Evidence Chain Verification</vois:proofName>
        </vois:proof>
      </extensionElements>
      <incoming>sf-to-evidence-verify</incoming>
      <outgoing>sf-to-verification-gate</outgoing>
    </serviceTask>

    <sequenceFlow id="sf-to-verification-gate" sourceRef="evt-005-evidence-verified" targetRef="ctl-004-verification-gate" />

    <!-- ctl-004: Adjudication Requires Verification (verification_check) -->
    <exclusiveGateway id="ctl-004-verification-gate" name="Verification OK?" gatewayDirection="Diverging">
      <extensionElements>
        <vois:control>
          <vois:controlType>verification_check</vois:controlType>
          <vois:controlledBy ref="auth-001" />
          <vois:governanceRules>
            <vois:rule>Verify all evidence chains before seal</vois:rule>
          </vois:governanceRules>
        </vois:control>
      </extensionElements>
      <incoming>sf-to-verification-gate</incoming>
      <outgoing>sf-verified-ok</outgoing>
      <outgoing>sf-verified-fail</outgoing>
    </exclusiveGateway>

    <sequenceFlow id="sf-verified-ok" name="Pass" sourceRef="ctl-004-verification-gate" targetRef="evt-006-adjudication">
      <conditionExpression xsi:type="tFormalExpression">${verified == true}</conditionExpression>
    </sequenceFlow>
    <sequenceFlow id="sf-verified-fail" name="Fail" sourceRef="ctl-004-verification-gate" targetRef="end-escalated" />

    <endEvent id="end-escalated" name="Escalated for Review">
      <incoming>sf-verified-fail</incoming>
    </endEvent>

    <!-- evt-006: Adjudication Sealed (sealing) -->
    <userTask id="evt-006-adjudication" name="Adjudication Sealed">
      <extensionElements>
        <vois:eventType value="sealing" />
        <vois:objectRef ref="obj-005" entityType="object" />
        <vois:authorityRef ref="auth-001" entityType="authority" />
        <vois:evidenceRefs>
          <vois:evidenceRef ref="prf-004" />
        </vois:evidenceRefs>
        <vois:proof>
          <vois:proofId>prf-005</vois:proofId>
          <vois:proofType>digital_seal</vois:proofType>
          <vois:proofName>Adjudication Seal</vois:proofName>
        </vois:proof>
      </extensionElements>
      <incoming>sf-verified-ok</incoming>
      <outgoing>sf-to-audit</outgoing>
    </userTask>

    <sequenceFlow id="sf-to-audit" sourceRef="evt-006-adjudication" targetRef="task-audit" />

    <!-- ctl-005: Mandatory Regulatory Copy (policy_enforcement) -->
    <serviceTask id="task-audit" name="Generate Audit Record">
      <extensionElements>
        <vois:eventType value="audit" />
        <vois:control>
          <vois:controlType>policy_enforcement</vois:controlType>
          <vois:controlledBy ref="auth-003" />
          <vois:governanceRules>
            <vois:rule>Audit record must be generated per claim</vois:rule>
          </vois:governanceRules>
        </vois:control>
        <vois:objectRef ref="obj-006" entityType="object" />
        <vois:proof>
          <vois:proofId>prf-006</vois:proofId>
          <vois:proofType>audit_log</vois:proofType>
          <vois:proofName>Compliance Audit Record</vois:proofName>
        </vois:proof>
      </extensionElements>
      <incoming>sf-to-audit</incoming>
      <outgoing>sf-end-insurer</outgoing>
    </serviceTask>

    <sequenceFlow id="sf-end-insurer" sourceRef="task-audit" targetRef="end-claim-processed" />

    <endEvent id="end-claim-processed" name="Claim Processed">
      <incoming>sf-end-insurer</incoming>
    </endEvent>
  </process>

  <!-- ================================================================== -->
  <!-- PROCESS: Independent Assessor (Pool: assessor)                     -->
  <!-- ================================================================== -->
  <process id="process-assessor" name="Assessor Process" isExecutable="false">
    <laneSet id="laneSet-assessor">
      <lane id="lane-assessor" name="Independent Assessor">
        <flowNodeRef>evt-004-report-sealed</flowNodeRef>
      </lane>
    </laneSet>

    <startEvent id="start-assessor" name="Assessment requested">
      <outgoing>sf-assessor-start</outgoing>
    </startEvent>

    <sequenceFlow id="sf-assessor-start" sourceRef="start-assessor" targetRef="evt-004-report-sealed" />

    <!-- evt-004: Assessor Report Sealed (sealing) -->
    <serviceTask id="evt-004-report-sealed" name="Assessor Report Sealed">
      <extensionElements>
        <vois:eventType value="sealing" />
        <vois:objectRef ref="obj-004" entityType="object" />
        <vois:authorityRef ref="auth-002" entityType="authority" />
        <vois:evidenceRefs>
          <vois:evidenceRef ref="prf-003" />
        </vois:evidenceRefs>
      </extensionElements>
      <incoming>sf-assessor-start</incoming>
      <outgoing>sf-assessor-end</outgoing>
    </serviceTask>

    <sequenceFlow id="sf-assessor-end" sourceRef="evt-004-report-sealed" targetRef="end-assessor" />

    <endEvent id="end-assessor" name="Report sent">
      <incoming>sf-assessor-end</incoming>
    </endEvent>
  </process>

  <!-- ================================================================== -->
  <!-- PROCESS: Regulator (Pool: regulator)                               -->
  <!-- ================================================================== -->
  <process id="process-regulator" name="Regulator Process" isExecutable="false">
    <laneSet id="laneSet-regulator">
      <lane id="lane-regulator" name="Regulator">
        <flowNodeRef>evt-007-audit</flowNodeRef>
      </lane>
    </laneSet>

    <startEvent id="start-regulator" name="Audit notification">
      <outgoing>sf-regulator-start</outgoing>
    </startEvent>

    <sequenceFlow id="sf-regulator-start" sourceRef="start-regulator" targetRef="evt-007-audit" />

    <serviceTask id="evt-007-audit" name="Regulatory Audit">
      <extensionElements>
        <vois:eventType value="audit" />
        <vois:objectRef ref="obj-006" entityType="object" />
        <vois:authorityRef ref="auth-003" entityType="authority" />
        <vois:evidenceRefs>
          <vois:evidenceRef ref="prf-006" />
        </vois:evidenceRefs>
      </extensionElements>
      <incoming>sf-regulator-start</incoming>
      <outgoing>sf-regulator-end</outgoing>
    </serviceTask>

    <sequenceFlow id="sf-regulator-end" sourceRef="evt-007-audit" targetRef="end-regulator" />

    <endEvent id="end-regulator" name="Audit complete">
      <incoming>sf-regulator-end</incoming>
    </endEvent>
  </process>

  <!-- ================================================================== -->
  <!-- BPMN DIAGRAM (Layout) — abbreviated; actual positions come from    -->
  <!-- the VOIS blueprint's nodes[].position values                       -->
  <!-- ================================================================== -->
  <bpmndi:BPMNDiagram id="BPMNDiagram_claim">
    <bpmndi:BPMNPlane id="BPMNPlane_claim" bpmnElement="collaboration-claim">
      <!-- Pool shapes -->
      <bpmndi:BPMNShape id="pool-claimant_di" bpmnElement="pool-claimant" isHorizontal="true">
        <omgdc:Bounds x="80" y="60" width="1400" height="260" />
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape id="pool-insurer_di" bpmnElement="pool-insurer" isHorizontal="true">
        <omgdc:Bounds x="80" y="340" width="1400" height="420" />
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape id="pool-assessor_di" bpmnElement="pool-assessor" isHorizontal="true">
        <omgdc:Bounds x="80" y="780" width="1400" height="260" />
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape id="pool-regulator_di" bpmnElement="pool-regulator" isHorizontal="true">
        <omgdc:Bounds x="80" y="1060" width="1400" height="260" />
      </bpmndi:BPMNShape>

      <!-- Lane shapes (insurer pool) -->
      <bpmndi:BPMNShape id="lane-intake_di" bpmnElement="lane-intake-agent" isHorizontal="true">
        <omgdc:Bounds x="200" y="340" width="1280" height="140" />
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape id="lane-verifier_di" bpmnElement="lane-evidence-verifier" isHorizontal="true">
        <omgdc:Bounds x="200" y="480" width="1280" height="140" />
      </bpmndi:BPMNShape>
      <bpmndi:BPMNShape id="lane-adjuster_di" bpmnElement="lane-adjuster" isHorizontal="true">
        <omgdc:Bounds x="200" y="620" width="1280" height="140" />
      </bpmndi:BPMNShape>

      <!-- Flow node shapes (positions from VOIS blueprint nodes) — abbreviated -->
      <!-- Edge waypoints — abbreviated -->
    </bpmndi:BPMNPlane>
  </bpmndi:BPMNDiagram>
</definitions>
```

---

## 5. TypeScript Libraries for BPMN XML Generation

### 5.1 Recommended: `@bpmnkit/core`

| Attribute | Value |
|---|---|
| **Package** | `@bpmnkit/core` |
| **License** | MIT |
| **TypeScript** | First-class, fully typed |
| **Dependencies** | Zero (for core) |
| **Install** | `npm install @bpmnkit/core` |
| **Repository** | https://github.com/bpmnkit/monorepo |
| **Docs** | https://bpmnkit.com/ |

**Features:**
- Fluent builder API: `Bpmn.createProcess(...).startEvent(...).serviceTask(...).endEvent(...).export()`
- Auto-layout via Sugiyama algorithm (no manual BPMNDI)
- Camunda 8/Zeebe compatible
- Parse, modify, and re-export BPMN XML
- Export to XML, DMN, forms

**Usage pattern:**

```typescript
import { Bpmn } from '@bpmnkit/core';

const { xml } = Bpmn.createProcess('claim-process', 'Claim Process')
  .startEvent('start-1', 'Claim Received')
  .serviceTask('svc-1', 'Verify Policy', { extensionElements: { ... } })
  .exclusiveGateway('gate-1', 'Policy Valid?')
  .branch('gate-1', 'valid', (flow) =>
    flow
      .userTask('task-1', 'Review Evidence')
      .endEvent('end-1', 'Approved')
  )
  .branch('gate-1', 'invalid', (flow) =>
    flow.endEvent('end-2', 'Rejected')
  )
  .export({ layout: true });
```

**Limitations for VOIS:**
- No native support for pools/lanes via the fluent builder (must add collaboration + participant elements manually or use BPMN model API)
- Custom extensions need to be injected into `extensionElements` programmatically
- `bpmnkit` targets Camunda 8 primarily; VOIS extensions require custom namespace handling

### 5.2 Alternative: `bpmn-moddle`

| Attribute | Value |
|---|---|
| **Package** | `bpmn-moddle` |
| **TypeScript** | Community types available |
| **Install** | `npm install bpmn-moddle` |
| **Repository** | https://github.com/bpmn-io/bpmn-moddle |

Lower-level than bpmnkit. Provides the BPMN meta-model for reading/writing XML directly.
Good for custom extension support but requires more manual work.

**Pattern:**

```typescript
import BpmnModdle from 'bpmn-moddle';

const moddle = new BpmnModdle();
const definitions = moddle.create('bpmn:Definitions', {
  targetNamespace: 'http://www.omg.org/spec/BPMN/20100524/MODEL',
  exporter: 'VOIS Simulator',
  exporterVersion: '0.1.0'
});
// ... build model programmatically
const { xml } = await moddle.toXML(definitions);
```

### 5.3 Alternative: `json-to-bpmn-xml`

Simple conversion from JSON node/edge model to BPMN XML. Suitable for direct mapping from VOIS simulator nodes + edges if no pools/lanes need to be generated.

### 5.4 Recommended Approach

**For VOIS: use `@bpmnkit/core` as the base, with `bpmn-moddle` as fallback for custom extension elements.**

The pipeline:

```
VOIS Blueprint JSON
       ↓
Normalizer (resolves refs, groups actors by trust_boundary)
       ↓
BPMN Model Builder (@bpmnkit/core or custom builder)
  - Creates collaboration with participants
  - Creates per-pool processes with lanes
  - Maps events → tasks with vois:extensions
  - Maps controls → gateways with vois:extensions
  - Adds dataObject references
  - Runs auto-layout
       ↓
Post-processor (injects vois:extension namespaces)
       ↓
Valid BPMN 2.0 XML
```

---

## 6. Key Pitfalls and Limitations

### 6.1 Structural Issues

| Pitfall | Impact | Mitigation |
|---|---|---|
| **BPMN requires each pool to have its own process** | A workflow with 4 trust boundaries generates 4 separate processes, each with start/end events | Message flows bridge the pools; ensure each process has valid internal flow |
| **Lanes can only contain flowNodeRefs** | Lanes cannot directly reference data objects, authorities, or proofs | Attach data/authority metadata via `extensionElements` on the tasks within lanes |
| **Cross-pool edges must be messageFlows, not sequenceFlows** | Sequence flows cannot cross pool boundaries | Convert all cross-trust-boundary edges to `<messageFlow>` elements |
| **BPMN has no native "possession" concept** | Object ownership tracking must be entirely in extensions | Use `vois:possession` extension on dataObjectReference + sequenceFlow |
| **BPMN has no native "authority" concept** | Authorities cannot be represented as flow nodes | Map to `<resource>` elements or define in extensions only; render outside collaboration |

### 6.2 Extension & Namespace Issues

| Pitfall | Impact | Mitigation |
|---|---|---|
| **Custom namespaces may not render in standard BPMN editors** | bpmn-js, Camunda Modeler, etc. will load the file but may not display VOIS extensions visually | Extensions appear in the XML source and properties panel; visual fidelity limited to standard BPMN shapes |
| **BPMN validators reject unknown extension namespaces** | Some tools (e.g. Camunda) will warn about unrecognised extensions | Use well-namespaced elements (`vois:`) and provide a VOIS XSD for validation |
| **`extensionElements` order matters** | Some parsers expect a specific order within `extensionElements` | Always place custom extensions AFTER standard vendor extensions (e.g. `camunda:`) |
| **Documentation inside extensions may confuse tools** | Tools may treat `vois:` documentation as rendering content | Use CDATA blocks and keep documentation at the definition level |

### 6.3 Lossy Mapping Concerns

| VOIS Concept | BPMN Representation | Fidelity |
|---|---|---|
| `actors[].trust_boundary` | Pool per boundary | ✅ High — pools are the correct BPMN construct |
| `agents[].capabilities` | Extension on serviceTask | ⚠️ Medium — semantics preserved in XML, not visible in diagram |
| `objects[].possessed_by` | Extension on dataObject | ⚠️ Medium — BPMN data objects don't model ownership |
| `objects[].lifecycle_transitions` | No direct mapping | ❌ Low — must be encoded as a state machine extension |
| `authorities[].seal_types` | Extension on resource | ⚠️ Medium — only visible as XML, not diagram |
| `proofs[]` | Extension on tasks + documentation | ⚠️ Medium — evidence chain is hard to visualise in BPMN |
| `risks[]` | No direct mapping | ❌ Low — could use documentation or a zip export |
| `edges[].type=possession` | Extension on sequenceFlow | ⚠️ Medium — edge type semantics lost on non-VOIS parsers |
| `edges[].type=delegation` | Extension on messageFlow | ⚠️ Medium |

### 6.4 BPMNDI (Diagram Layout) Pitfalls

| Pitfall | Impact | Mitigation |
|---|---|---|
| **BPMNDI coordinates are absolute pixels** | Layout must be computed dynamically from node positions | Map `nodes[].position` to `Bounds` with offset/scaling |
| **Lane labels require extra BPMNDI shapes** | Each lane and pool needs a named BPMNShape | Generate shapes for all pools and lanes automatically |
| **Waypoints for edges must be explicit** | Edge routing is not auto-derived from source/target bounds | Compute intermediate waypoints from edge source/target positions |
| **Auto-layout tools may not support custom extensions** | Layout algorithms don't account for extension element sizes | Accept auto-layout for standard BPMN flow; overrides for extensions |

### 6.5 Recommendations for Implementation

1. **Use `@bpmnkit/core` for structure + auto-layout**, then inject `vois:` extensions as a post-processing step.
2. **Create a VOIS XSD** and host it at a stable URL for validation tooling.
3. **Generate one BPMN file per trust boundary** if the workflow is very complex, OR a single collaboration file (recommended for most VOIS blueprints).
4. **Strip VOIS extensions for "legacy BPMN" export** — provide an option to exclude the `vois:` namespace for tools that can't handle unknown extensions.
5. **Always validate output** against the BPMN 2.0 XSD before claiming success. Many modelling tools will silently fail on structurally invalid XML.
6. **Consider BPMN + companion JSON** — for full fidelity, export the BPMN XML for process modelling and a companion `vois-meta.json` that carries all VOIS-specific semantics that BPMN can't represent natively.

### 6.6 Renderer Compatibility

| Tool | Extensions | Pools/Lanes | BPMNDI | Likely VOIS Compatibility |
|---|---|---|---|---|
| **Camunda Modeler** | Yes (`extensionElements`) | Full | Full | ⭐ High — will render pools/lanes; extensions in properties panel |
| **bpmn-js** | Yes | Full | Full | ⭐ High — extensible via custom renderers |
| **Flowable/Activiti** | Through vendor namespace | Full | Partial | ⚠️ Medium — needs `flowable:` or `activiti:` NS adaptation |
| **Signavio** | Limited | Full | Full | ⚠️ Low — may strip unknown namespaces |
| **Lucidchart** | No | Full | Via import | ❌ Low — visual only, loses extensions |
| **Visio** | No | Full | Via import | ❌ Low — visual only |
| **Bizagi** | Limited | Full | Full | ⚠️ Medium |
| **Generic XML tools** | Full (XML) | N/A | N/A | ⭐ High — all extension data preserved in source |

---

## Appendix: Quick-Reference Mapping Matrix

| VOIS Concept | BPMN Element | Extension? | Fidelity | Notes |
|---|---|---|---|---|
| Blueprint name | `<collaboration name>` | — | ✅ | |
| Actor | `<lane>` (within `<laneSet>`) | `vois:actorType` | ✅ | |
| Trust boundary | `<participant>` (pool) | — | ✅ | |
| Agent | `<serviceTask>` | `vois:delegatedBy`, `vois:capabilities` | ⚠️ | |
| Digital Object | `<dataObject>` + `<dataObjectReference>` | `vois:vaulted`, `vois:possession` | ✅ | |
| Authority | `<resource>` | `vois:authority` | ⚠️ | Not a flow node |
| Event (creation) | `<startEvent>` or `<task>` | `vois:eventType` | ✅ | |
| Event (sealing) | `<serviceTask>` | `vois:eventType="sealing"` | ✅ | |
| Event (verification) | `<businessRuleTask>` or `<serviceTask>` | `vois:eventType="verification"` | ✅ | |
| Event (audit) | `<serviceTask>` | `vois:eventType="audit"` | ✅ | |
| Control (gate) | `<exclusiveGateway>` | `vois:control.type` | ✅ | |
| Control (policy) | `<businessRuleTask>` | `vois:control.type` | ✅ | |
| Proof | `<documentation>` + extension | `vois:proof` | ⚠️ | Attached to producing task |
| Sequence flow (intra-pool) | `<sequenceFlow>` | `vois:edgeType` | ✅ | |
| Cross-pool flow | `<messageFlow>` | `vois:edgeType` | ✅ | |
| Data flow | `<dataInputAssociation>` / `<dataOutputAssociation>` | — | ✅ | |
| Node position | BPMNDI `<Bounds>` | — | ✅ | |
| Risk | None | `vois:risk` on definitions | ❌ | Companion JSON needed |
| Lifecycle transitions | None | `vois:lifecycleState` on dataObject | ❌ | |

---

*Generated by the Vaulted Objects Infrastructure Simulator Research Pipeline*
*Source: vaulted-context/research/bpmn-export.md*
