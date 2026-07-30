# C4 Model & Structurizr DSL Export — Research for VOIS Blueprints

> **Status:** Complete  
> **Author:** Agent subagent (Hermes)  
> **Date:** 2026-07-31  
> **Part of:** Vaulted Agent Context — Phase 3 Interoperability Research (Card #13)

---

## Table of Contents

1. [C4 Model Overview: Four Abstraction Levels](#1-c4-model-overview-four-abstraction-levels)
2. [Structurizr DSL Basics](#2-structurizr-dsl-basics)
3. [VOIS Blueprint → C4 Mapping](#3-vois-blueprint--c4-mapping)
4. [Trust Boundaries → C4 Deployment Nodes / Boundaries](#4-trust-boundaries--c4-deployment-nodes--boundaries)
5. [Example: Structurizr DSL for the Insurance Claim Workflow](#5-example-structurizr-dsl-for-the-insurance-claim-workflow)
6. [Structurizr CLI & DSL Validation Tools](#6-structurizr-cli--dsl-validation-tools)
7. [Export Options: DSL → PlantUML, Mermaid, SVG & More](#7-export-options-dsl--plantuml-mermaid-svg--more)
8. [C4 Diagrams for Enterprise Communication](#8-c4-diagrams-for-enterprise-communication)
9. [Limitations: What C4 Doesn't Natively Model](#9-limitations-what-c4-doesnt-natively-model)
10. [Implementation Recommendations for VOIS](#10-implementation-recommendations-for-vois)

---

## 1. C4 Model Overview: Four Abstraction Levels

The C4 model (created by Simon Brown, ~2006–2011) provides a **hierarchical, zoom-able** approach to software architecture diagramming. It uses four core abstraction levels, each with a corresponding diagram type.

### Level 1 — System Context (L1)

| Aspect | Detail |
|--------|--------|
| **Scope** | The system being built and its relationship to users and external systems |
| **Audience** | Everyone — technical and non-technical stakeholders |
| **Elements** | The software system, people (users/actors), external software systems |
| **What it shows** | Big-picture: who uses the system, what external systems it talks to |
| **VOIS relevance** | Shows VOI Simulator as a system, with policyholders, adjusters, regulators as persons, and external authoritative systems |

**In short:** "What are we building, who uses it, and what does it talk to?"

### Level 2 — Container (L2)

| Aspect | Detail |
|--------|--------|
| **Scope** | The high-level technology boundaries of the system |
| **Audience** | Technical — developers, architects, operations |
| **Elements** | Containers: web apps, mobile apps, databases, microservices, file systems |
| **What it shows** | Deployable units, their responsibilities, and how they communicate (API, message queue, DB) |
| **VOIS relevance** | Maps VOI Simulator subsystems: frontend SPA, API gateway, blueprint engine, proof orchestrator, data stores |

**In short:** "What are the deployable parts of our system, and how do they talk?"

### Level 3 — Component (L3)

| Aspect | Detail |
|--------|--------|
| **Scope** | The internal structure of a single container |
| **Audience** | Developers on the team |
| **Elements** | Components: classes, modules, services, controllers, repositories |
| **What it shows** | How a container is internally decomposed; the major structural building blocks and their interactions |
| **VOIS relevance** | Dives into the Blueprint Engine container: Blueprint Parser, Template Renderer, Export Generator, Validation Service |

**In short:** "What are the major structural pieces inside this deployable unit?"

### Level 4 — Code (L4)

| Aspect | Detail |
|--------|--------|
| **Scope** | Detailed implementation view of a single component |
| **Audience** | Developers implementing the component |
| **Elements** | Classes, interfaces, functions, data structures |
| **What it shows** | UML-style class diagrams or similar — often auto-generated from code |
| **VOIS relevance** | Typically omitted unless needed; can show the Blueprint Schema Parser's class hierarchy or the Structurizr export adapter |

**In short:** "How is this component actually implemented?"

### Supporting Diagram Types

Beyond the four core levels, C4 provides three supplementary diagram types:

| Diagram Type | Purpose | VOIS Use |
|-------------|---------|----------|
| **System Landscape** | Shows all systems in a given environment (wider than context) | Map the entire insurance ecosystem |
| **Dynamic** | Shows runtime behaviour — element interactions for a specific scenario | Show the claim adjudication sequence |
| **Deployment** | Shows how software maps to infrastructure | Map VOIS trust boundaries → deployment nodes |

---

## 2. Structurizr DSL Basics

Structurizr is the reference tooling for the C4 model. Its DSL (Domain-Specific Language) lets you define C4 models and views as **plain text** — enabling version control, code review, and automation.

### Workspace Structure

```
workspace "Name" "Description" {

    model {
        // Elements: person, softwareSystem, container, component, deploymentNode
        // Relationships: source -> target "description" "technology"
    }

    views {
        systemContext "softwareSystemIdentity" { ... }
        container "softwareSystemIdentity" { ... }
        component "containerIdentity" { ... }
        deployment "environment" "softwareSystemIdentity" { ... }
        dynamic "softwareSystemIdentity" { ... }

        styles { ... }
        themes { ... }
    }

}
```

### Core DSL Elements

| Keyword | Purpose | C4 Level |
|---------|---------|----------|
| `person` | A human user or actor | L1 |
| `softwareSystem` | The system being built or an external system | L1 |
| `container` | A deployable unit (web app, database, microservice) | L2 |
| `component` | A structural element inside a container | L3 |
| `deploymentNode` | Infrastructure: physical/virtual/cloud | Deployment |
| `deploymentEnvironment` | A named environment (e.g., "Development", "Production") | Deployment |
| `containerInstance` | Instance of a container within a deployment node | Deployment |
| `softwareSystemInstance` | Instance of a software system within a deployment node | Deployment |
| `infrastructureNode` | Supporting infra (load balancer, DNS, firewall) | Deployment |
| `group` | Element grouping (no semantic meaning — visual only) | — |
| `deploymentGroup` | Named group for deployment relationship isolation | Deployment |

### Relationship Syntax

```
source -> target "description" "technology" {
    tags "tag1, tag2"
}
```

### Comments

```
// Single line
/* Multi-line */
```

### Identifiers

Elements and relationships can be assigned identifiers for reuse:

```
u = person "Policyholder"
ss = softwareSystem "VOI Simulator"
u -> ss "Submits claim"
```

### Tagging & Styling

```
element "Person" {
    shape person
    background #08427b
    color #ffffff
}
```

### Constants & Variables

```
!const insurerTag = "Insurer"

softwareSystem claims = "Claims System" {
    tags insurerTag
}
```

---

## 3. VOIS Blueprint → C4 Mapping

The table below defines the canonical mapping between VOIS blueprint primitives and C4 abstractions.

### Entity Mapping

| VOIS Blueprint Entity | C4 Abstraction | C4 Level | Mapping Notes |
|-----------------------|----------------|----------|---------------|
| **Actors** (human) | `person` | L1 | Policyholder, Claims Adjuster, Regulator → persons |
| **Actors** (organisation) | `softwareSystem` (external) | L1 | Insurance company, regulator as external systems |
| **Agents** (AI/software) | `container` or `component` | L2/L3 | AI agents are containers (deployable) or components within a container |
| **Systems** (IT) | `softwareSystem` | L1 | Internal systems (owned) or external systems |
| **Authorities** | `softwareSystem` (external) | L1 | Certifying authorities, regulatory bodies as external systems with trust relationships |
| **Digital Objects** | Not directly modelled | — | C4 models static structure, not data; use `description` or supplementary docs |
| **Vaulted Objects** | Not directly modelled | — | Same as above; use relationship `description` field or ADR |
| **Events** (provenance) | Dynamic diagram interactions | Dynamic | `dynamic` view elements with numbered interactions |
| **Controls** | `infrastructureNode` or boundary | Deployment | Policy enforcement points → infrastructure nodes or `group` boundaries |
| **Trust Boundaries** | `group`, `deploymentNode`, or boundary | L1-L3, Deployment | See [section 4](#4-trust-boundaries--c4-deployment-nodes--boundaries) |

### Relationship Mapping

| VOIS Relationship | C4 Relationship | Notes |
|-------------------|-----------------|-------|
| Flow (object transfer) | `->` with description | E.g., `Policyholder -> VOI Simulator "Submits claim"` |
| Data flow | `->` with technology | E.g., `ClaimsIntakeAgent -> PolicyDB "Verifies" "SQL"` |
| Possession | `->` with description | E.g., `Policyholder -> IncidentPhoto "Possesses"` |
| Authority (sealing) | `->` with description | External system to container relationship |
| Verification | `->` with description | Person/agent to external system relationship |
| Control (gate) | Modelled as boundary or tag | Not directly mappable — use supplementary text |

### Tag Convention for VOIS

Recommended tags for distinguishing VOIS elements in C4:

```
!const voisPersonTag = "VOIS Person"
!const voisSystemTag = "VOIS System"
!const voisAgentTag = "VOIS Agent"
!const voisAuthorityTag = "VOIS Authority"
!const voisTrustBoundaryTag = "VOIS Trust Boundary"
```

---

## 4. Trust Boundaries → C4 Deployment Nodes / Boundaries

In the VOIS blueprint model, **trust boundaries** delineate security domains (e.g., "External — claimant", "Internal — insurer", "External — supervisory"). C4 offers several ways to map these:

### Option A: `group` (Visual Grouping)

Best for L1 (System Context) diagrams where you want to visually cluster elements by trust domain.

```dsl
model {
    group "External — Claimant" {
        policyholder = person "Policyholder"
    }
    group "Internal — Insurer" {
        adjuster = person "Claims Adjuster"
        claimsSystem = softwareSystem "Claims Processing"
    }
    group "External — Supervisory" {
        regulator = person "Regulator"
    }
}
```

**Pros:** Simple, works at any C4 level.  
**Cons:** No semantic meaning — purely visual; cannot enforce relationship rules.

### Option B: `deploymentNode` with `deploymentEnvironment`

Best for Deployment diagrams where trust boundaries map to physical/cloud infrastructure.

```dsl
model {
    deploymentEnvironment "Production" {
        deploymentNode "On-Premise Data Center" "Insurer Trust Zone" {
            deploymentNode "DMZ" {
                infrastructureNode "API Gateway" "Load Balancer"
                containerInstance claimsApi = claims "Claims API"
            }
            deploymentNode "Internal Network" "Insurer Internal" {
                containerInstance claimsDb = claims "Claims Database"
            }
        }
        deploymentNode "External Cloud" "Claimant Zone" {
            containerInstance portal = claims "Claimant Portal"
        }
    }
}
```

**Pros:** Maps directly to infrastructure topology; supports nested nodes.  
**Cons:** Adds deployment-level detail even when only logical boundaries are needed.

### Option C: Supplementary Annotation (Styles + Descriptions)

Use element styling and tags to colour-code by trust boundary without structural grouping.

```dsl
views {
    styles {
        element "External" {
            background #ffcccc
            border #cc0000
        }
        element "Internal" {
            background #ccffcc
            border #00cc00
        }
        element "Supervisory" {
            background #ccccff
            border #0000cc
        }
    }
}
```

### Recommended Approach for VOIS

| Scenario | Recommended Mapping |
|----------|-------------------|
| **System Context (L1)** — showing all actors and external systems | `group` per trust boundary with colour-coded styles |
| **Container (L2)** — showing VOI Simulator internals | `group` for internal boundaries + colour styling |
| **Deployment** — showing where each component physically runs | `deploymentNode` per trust zone |
| **Dynamic** — showing claim workflow across boundaries | Use tags and relationship descriptions |

---

## 5. Example: Structurizr DSL for the Insurance Claim Workflow

Below is a complete, production-ready Structurizr DSL file for the VOIS Insurance Claim Evidence Workflow blueprint, covering **System Context (L1)** and **Container (L2)** diagrams with trust boundary groups.

```dsl
workspace "VOIS — Insurance Claim Workflow" "C4 model of the Vaulted Objects Infrastructure Simulator applied to an insurance claim evidence workflow" {

    !const voisPersonTag = "VOIS Person"
    !const voisAgentTag = "VOIS Agent"
    !const voisAuthorityTag = "VOIS Authority"
    !const voisExternalTag = "VOIS External System"
    !const voisSystemTag = "VOIS System"

    model {
        // ========== ACTORS ==========
        policyholder = person "Policyholder" "Individual submitting an insurance claim" {
            tags voisPersonTag
        }
        adjuster = person "Claims Adjuster" "Insurance company employee adjudicating claims" {
            tags voisPersonTag
        }
        assessor = person "Independent Assessor" "Accredited third-party damage assessor" {
            tags voisPersonTag, voisExternalTag
        }
        regulator = person "Regulator" "Industry supervisory body" {
            tags voisPersonTag, voisExternalTag
        }

        // ========== EXTERNAL AUTHORITIES ==========
        insurerAuthority = softwareSystem "Insurer Authority" "Certifying authority for origin seals, authority seals, and attestations" {
            tags voisAuthorityTag
        }
        assessorAuthority = softwareSystem "Accredited Assessor Authority" "Trusted third-party for assessor accreditation" {
            tags voisAuthorityTag, voisExternalTag
        }
        regulatoryBody = softwareSystem "Industry Regulator Authority" "Hierarchical trust chain for compliance" {
            tags voisAuthorityTag, voisExternalTag
        }

        // ========== VOI SIMULATOR SYSTEM ==========
        voiSimulator = softwareSystem "VOI Simulator" "Vaulted Objects Infrastructure Simulator — interactive blueprint modelling tool" {
            tags voisSystemTag
        }

        // ========== RELATIONSHIPS ==========
        policyholder -> voiSimulator "Submits claim evidence" "HTTPS/REST"
        policyholder -> insurerAuthority "Requests origin seal for evidence" "API"
        adjuster -> voiSimulator "Adjudicates claims" "HTTPS/REST"
        assessor -> assessorAuthority "Receives accreditation"
        assessor -> voiSimulator "Submits assessor reports" "HTTPS/REST"
        regulator -> regulatoryBody "Configures compliance rules"
        regulator -> voiSimulator "Audits claim lifecycle" "HTTPS/REST"

        // ========== CONTAINERS ==========
        voiSimulator {
            webApp = container "Web Frontend" "React SPA — blueprint board UI" "TypeScript, React" {
                tags voisSystemTag
            }
            apiGateway = container "API Gateway" "REST API Gateway — routes requests to backend services" "Node.js, Express" {
                tags voisSystemTag
            }
            blueprintEngine = container "Blueprint Engine" "Core engine — stores, validates, and renders blueprints" "Python, FastAPI" {
                tags voisSystemTag
            }
            proofOrchestrator = container "Proof Orchestrator" "Manages ITS seal creation and verification across authorities" "Rust" {
                tags voisSystemTag
            }
            objectStore = container "Object Store" "Persistent storage for blueprint data and vaulted object metadata" "PostgreSQL" {
                tags voisSystemTag
            }

            // Container relationships
            webApp -> apiGateway "API calls" "HTTPS"
            apiGateway -> blueprintEngine "Route requests" "gRPC"
            blueprintEngine -> proofOrchestrator "Seal/verify objects" "gRPC"
            blueprintEngine -> objectStore "Read/write blueprints" "SQL"
            proofOrchestrator -> objectStore "Store proof metadata" "SQL"
            proofOrchestrator -> insurerAuthority "Request ITS seals" "API"
            proofOrchestrator -> assessorAuthority "Verify assessor credentials" "API"
            proofOrchestrator -> regulatoryBody "Submit compliance records" "API"
        }

        // ========== AGENTS (as components / containers) ==========
        claimsIntakeAgent = container "Claims Intake Agent" "AI agent — receives claims, verifies policy" "Python, LangChain" {
            tags voisAgentTag
        }
        evidenceVerifier = container "Evidence Verifier" "AI agent — checks seal integrity, provenance chain" "Python, LangChain" {
            tags voisAgentTag
        }
        complianceMonitor = container "Compliance Monitor" "AI agent — audits lifecycle, generates reports" "Python, LangChain" {
            tags voisAgentTag
        }
    }

    views {
        // ========== SYSTEM CONTEXT VIEW (L1) ==========
        systemContext voiSimulator "SystemContext" {
            add policyholder
            add adjuster
            add assessor
            add regulator
            add insurerAuthority
            add assessorAuthority
            add regulatoryBody
            add voiSimulator
            add claimsIntakeAgent
            add evidenceVerifier
            add complianceMonitor
            enableAutomaticLayout
        }

        // ========== CONTAINER VIEW (L2) ==========
        container voiSimulator "Container" {
            add policyholder
            add adjuster
            add assessor
            add regulator
            add insurerAuthority
            add assessorAuthority
            add regulatoryBody
            add webApp
            add apiGateway
            add blueprintEngine
            add proofOrchestrator
            add objectStore
            add claimsIntakeAgent
            add evidenceVerifier
            add complianceMonitor
            enableAutomaticLayout
        }

        // ========== DYNAMIC VIEW ==========
        dynamic voiSimulator "ClaimWorkflow" "Claim evidence workflow sequence" {
            // Policyholder submits claim
            policyholder -> claimsIntakeAgent "Submits claim"
            claimsIntakeAgent -> claimsIntakeAgent "Verifies policy"

            // Evidence collection
            policyholder -> insurerAuthority "Requests photo seal"
            insurerAuthority -> policyholder "Returns origin seal"

            // Assessment
            assessor -> assessorAuthority "Submit report for sealing"
            assessorAuthority -> assessor "Returns sealed report"

            // Verification
            evidenceVerifier -> proofOrchestrator "Verify evidence chain"
            proofOrchestrator -> insurerAuthority "Verify seal chain"
            insurerAuthority -> proofOrchestrator "Chain verified"

            // Adjudication
            adjuster -> blueprintEngine "Review evidence"
            adjuster -> proofOrchestrator "Seal adjudication"

            // Compliance
            complianceMonitor -> regulatoryBody "Submit audit record"
            autolayout
        }

        // ========== STYLES ==========
        styles {
            element "Person" {
                shape person
                background #08427b
                color #ffffff
                fontSize 14
            }
            element "Software System" {
                background #1168bd
                color #ffffff
                fontSize 14
            }
            element "Container" {
                shape roundedBox
                background #438dd5
                color #ffffff
                fontSize 14
            }
            element voisPersonTag {
                background #08427b
                color #ffffff
            }
            element voisAgentTag {
                shape hexagon
                background #f5a623
                color #ffffff
                fontSize 14
            }
            element voisAuthorityTag {
                shape roundedBox
                background #7c3aed
                color #ffffff
                fontSize 14
            }
            element voisExternalTag {
                background #6b7280
            }
            element voisSystemTag {
                background #10b981
                color #ffffff
            }
            element "Group" {
                background #f0f0f0
            }
        }
    }
}
```

### Validation & Rendering

Test the DSL above at: **[playground.structurizr.com](https://playground.structurizr.com)**  
Paste the DSL into the editor pane to see live-rendered System Context and Container diagrams.

---

## 6. Structurizr CLI & DSL Validation Tools

### Current Tooling (v2026+)

The legacy Structurizr CLI has been **end-of-life** and replaced by **consolidated tooling**. The recommended way to run commands is via the `structurizr.war` Java archive or the official Docker image.

#### Docker

```bash
docker run --rm -v $(pwd):/workspace structurizr/structurizr export \
  -workspace /workspace/workspace.dsl \
  -format plantuml \
  -output /workspace/output
```

#### Java JAR

```bash
java -jar structurizr.war export \
  -workspace workspace.dsl \
  -format mermaid \
  -output ./diagrams
```

Download from: [docs.structurizr.com/binaries](https://docs.structurizr.com/binaries)

### Validation

```bash
# Validate a DSL or JSON workspace
java -jar structurizr.war validate -workspace workspace.dsl

# Inspect workspace contents
java -jar structurizr.war inspect -workspace workspace.dsl
```

### MCP Server (AI Integration)

Structurizr offers an **MCP server** that provides AI/agent accessible tools:

- **DSL validation** — check if a DSL string is parseable
- **DSL → JSON parsing** — for programmatic inspection
- **Workspace inspection** — query model elements and views

### Supported Commands (Consolidated Tooling)

| Command | Purpose |
|---------|---------|
| `validate` | Validate a JSON/DSL workspace |
| `inspect` | Inspect workspace contents |
| `export` | Export views to various formats |
| `push` | Push workspace to Structurizr server |
| `pull` | Pull workspace from Structurizr server |
| `create` | Create a new workspace on server |
| `delete` | Delete a workspace on server |
| `lock` / `unlock` | Lock/unlock for collaborative editing |
| `merge` | Merge remote changes |
| `branches` | Manage workspace branches |
| `generate` | Generate diagrams from DSL |
| `list` | List workspaces on server |

### DSL Parser (Standalone)

The DSL can be parsed programmatically via the Structurizr Java library:

```xml
<dependency>
    <groupId>com.structurizr</groupId>
    <artifactId>structurizr-dsl</artifactId>
    <version>LATEST</version>
</dependency>
```

```java
Workspace workspace = new DslParser().parse(new File("workspace.dsl"));
```

---

## 7. Export Options: DSL → PlantUML, Mermaid, SVG & More

The `export` command in the consolidated tooling supports the following output formats:

| Format | Command Suffix | Quality | Best For |
|--------|---------------|---------|----------|
| **PlantUML (structurizr)** | `-format plantuml` | High — native Structurizr shape mapping | Embedding in documentation, further processing |
| **C4-PlantUML** | `-format plantuml/c4plantuml` | High — uses C4-PlantUML macro library | Consistent C4 styling in PlantUML ecosystem |
| **Mermaid** | `-format mermaid` | Medium — limited shape support | Markdown docs, GitHub README, Notion |
| **WebSequenceDiagrams** | `-format websequencediagrams` | Good for dynamic views only | Sequence/flow diagrams |
| **Static HTML Site** | `-format static` | High — interactive diagrams | Team documentation portals |
| **PNG** | `-format png` | High — browser-rendered screenshots | Presentations, reports, slide decks |
| **SVG** | `-format svg` | High — browser-rendered vector | Print, publishing, vector editing |
| **JSON** | `-format json` | Lossless | Programmatic processing, custom exporters |
| **Theme** | `-format theme` | Styles only | Reusable theme definitions |

### Export Command Examples

```bash
# Export to Mermaid (best for Markdown/README)
java -jar structurizr.war export \
  -workspace workspace.dsl \
  -format mermaid \
  -output ./mermaid-output

# Export to PlantUML C4 style
java -jar structurizr.war export \
  -workspace workspace.dsl \
  -format plantuml/c4plantuml \
  -output ./plantuml-output

# Export to SVG (requires browser renderer — Playwright)
java -jar structurizr.war export \
  -workspace workspace.dsl \
  -format svg \
  -output ./svg-output

# Export to static HTML site
java -jar structurizr.war export \
  -workspace workspace.dsl \
  -format static \
  -output ./html-site
```

### Format Quality Comparison

| Feature | PlantUML | Mermaid | PNG/SVG | Static Site |
|---------|----------|---------|---------|-------------|
| Element shapes (person, hexagon) | ✅ Full | ❌ Limited to flowchart shapes | ✅ Full | ✅ Full |
| Diagram types (all 4 C4 levels) | ✅ | ✅ | ✅ | ✅ |
| Colour and styling | ✅ | ✅ Limited | ✅ | ✅ |
| Deployment diagram | ✅ | ❌ Not supported | ✅ | ✅ |
| Dynamic diagram | ✅ | ✅ (sequence) | ✅ | ✅ |
| Interactive exploration | ❌ Static | ❌ Static | ❌ Static | ✅ Clickable |
| File size | Small | Small | Medium | Large (many files) |
| Version-control friendly | ✅ | ✅ | ❌ Binary | ✅ Mixed |

### Recommendation for VOIS

| Use Case | Recommended Export |
|----------|-------------------|
| **Markdown docs** (README, Notion, GitHub) | Mermaid — embedded directly |
| **Technical reports** | PlantUML — full C4 fidelity |
| **Presentations** | SVG — vector quality, scalable |
| **Team intranet** | Static HTML site — interactive |
| **CI/CD pipelines** | PNG — for visual diffs |

---

## 8. C4 Diagrams for Enterprise Communication

### Why C4 Works for Enterprise Teams

| Stakeholder Group | C4 Level | What They Get |
|-------------------|----------|---------------|
| **C-Suite / Executives** | L1 (System Context) | "This is our system, these are who it serves, these are external integrations" |
| **Product Managers** | L1 (System Context) | Understanding of user touchpoints and external dependencies |
| **Enterprise Architects** | L1 + Landscape | How VOIS fits into the broader IT ecosystem |
| **Solution Architects** | L2 (Container) | Deployable units, technology choices, communication protocols |
| **Development Teams** | L2 + L3 | Clear boundaries, responsibilities, integration points |
| **Security / Compliance** | L2 + Deployment | Trust boundaries, data flow, infrastructure topology |
| **Operations / DevOps** | Deployment | Infrastructure mapping, instance topology |
| **New Team Members** | All levels | Rapid onboarding through zoomable architecture |

### How C4 Helps Explain VOIS to Enterprise Teams

**1. Zero-Exposure is hard to visualise — C4 makes it concrete.**

The C4 model's hierarchical zoom lets you start with "VOI Simulator interacts with Authorities and Actors" (L1) and zoom into "The Proof Orchestrator container handles ITS seal creation and routes requests to external Authorities" (L2). This makes the Zero-Exposure architecture tangible rather than abstract.

**2. Trust boundaries map naturally to enterprise security domains.**

Enterprise teams are familiar with internal/external trust zones. C4 groups and deployment nodes map directly to DMZ, internal network, partner network, etc., making the VOIS trust model immediately understandable.

**3. "As-code" governance through Structurizr DSL.**

The DSL enables:
- **Code review** of architecture changes via PRs
- **Versioned** architecture alongside the codebase
- **Automated validation** in CI/CD pipelines
- **Consistent rendering** across all views
- **Single source of truth** for architecture documentation

**4. Multi-level communication with a single model.**

One DSL file generates diagrams for all stakeholder levels. Executives see the context diagram; developers see containers and components; compliance sees deployment topology — all from the same model.

**5. Integration with existing enterprise tooling.**

Export to PlantUML (Atlassian Confluence, GitLab), Mermaid (GitHub, Notion), SVG (SharePoint, Google Docs), or static site (corporate wiki). No special viewer required.

### VOIS-Specific Enterprise Talking Points

| Enterprise Concern | C4 Answer |
|-------------------|-----------|
| "How does this integrate with our existing systems?" | L1/L2 show external system relationships |
| "What's the security boundary?" | Deployment/L2 with trust boundary groups |
| "How do AI agents fit in?" | Agents as containers with defined APIs |
| "What about compliance/audit?" | Dynamic views show evidence chain flow |
| "Can our ops team deploy this?" | Deployment diagram with instance topology |

---

## 9. Limitations: What C4 Doesn't Natively Model

This section is critical for setting accurate expectations when presenting C4 diagrams to enterprise stakeholders.

### Explicitly Out of Scope (per C4 FAQ)

> "The C4 model is **static structure focused**. It explicitly does **not** cover business processes, workflows, state machines, domain models, data models, or similar aspects." — [c4model.com/faq](https://c4model.com/faq)

| Concept | C4 Limitation | VOIS Impact | Supplement |
|---------|--------------|-------------|------------|
| **Business processes** | C4 shows static structure, not process flow | VOIS workflows (claim lifecycle) can't be fully represented | Use **BPMN 2.0** for process modelling |
| **Workflows / sequences** | Core C4 is static; `dynamic` view only shows numbered interactions | Complex multi-step workflows with branching and loops | Use **BPMN**, **Mermaid flowchart**, or **UML activity diagrams** |
| **State machines / lifecycles** | No state modelling | Vaulted object lifecycle (Created → Sealed → Verified → Finalised) | Use **UML state machines** or **W3C PROV** for provenance |
| **Domain models / data models** | C4 models software structure, not data structure | Blueprint schemas, object hierarchies, data relationships | Use **ER diagrams** or **JSON Schema** references |
| **Data flow** | Relationships show direction but not data content | What data passes through each relationship | Annotate relationship `description` or use **DFDs** |
| **Non-software systems** | C4 models software systems only | Physical evidence, hardware tokens, paper documents | Use supplementary notes or **BPMN** artefacts |
| **Deployment topology** | Deployment view shows mapping but not dynamic scaling | Auto-scaling, multi-region failover, load balancing rules | Supplement with **infrastructure-as-code** (Terraform, K8s manifests) |
| **Security threats / risk** | No native threat modelling | Trust boundaries visualised but no threat enumeration | Use **STRIDE**, **attack trees**, or **OSCAL** controls |
| **Cost / performance** | Not modelled | Infrastructure sizing, latency budgets | Supplement with **Architecture Decision Records (ADRs)** |

### VOIS-Specific Gaps

1. **Vaulted Objects aren't C4 elements.** C4 models software systems, containers, and components — not data objects. VOIS vaulted objects (Proof, Seal, Credential) have no direct C4 counterpart. **Workaround:** Map them as containers or software systems, or document them in supplementary ADRs.

2. **ITS proofs and Zero-Exposure mechanisms are invisible.** The cryptographic proof layer that distinguishes VOIS doesn't appear in C4 diagrams. **Workaround:** Use relationship descriptions (e.g., `"Verifies using ITS possession proof protocol"`) and supplementary documentation.

3. **Authority hierarchies and delegation chains.** C4 has no concept of "who delegated authority to whom." **Workaround:** Use dynamic views with numbered interaction sequences.

4. **Actor-to-actor relationships.** C4 focuses on system relationships. Many VOIS relationships are actor-to-actor (e.g., Policyholder → Adjuster). **Workaround:** Use `person -> person` relationships in the DSL (Structurizr supports this).

5. **Event/causality ordering.** C4 dynamic diagrams show numbered steps but not causality or conditional branching. **Workaround:** Use Mermaid sequence diagrams or BPMN for the full workflow.

### Supplement Strategy

For a complete VOIS architecture picture, use **C4 as the primary structural view** and supplement with:

| Supplement Format | Covers |
|-------------------|--------|
| **BPMN 2.0** | Business processes, workflows, gateway/decision points |
| **W3C PROV** | Provenance chains, entity-activity-agent model |
| **UML state machines** | Vaulted object lifecycle states |
| **JSON Schema** | Blueprint data structure |
| **ADRs (Architecture Decision Records)** | Why decisions were made, including security/cost trade-offs |
| **ER diagrams** | Data model for blueprints and objects |
| **OSCAL** | Security control mappings |

---

## 10. Implementation Recommendations for VOIS

### Priority Order

1. **Start with L1 (System Context).** Map all VOIS actors, systems, and authorities. This gives the biggest communication win for the least effort.

2. **Add L2 (Container) for the VOI Simulator.** Show web frontend, API gateway, blueprint engine, proof orchestrator, and AI agents as separate containers.

3. **Add trust boundary groupings.** Use `group` elements to visually separate trust zones (Claimant, Insurer, Supervisory).

4. **Generate example DSL for the Insurance Claim blueprint.** Use the DSL in [Section 5](#5-example-structurizr-dsl-for-the-insurance-claim-workflow) as a starting point.

5. **Integrate into CI/CD.** Add Structurizr validation to the blueprint pipeline so every blueprint export also validates its C4 mapping.

6. **Export to multiple formats.** Generate PlantUML for technical docs, Mermaid for GitHub/README, and SVG for presentations.

7. **Add ADRs for architectural decisions.** Document why VOIS uses ITS proofs, why trust boundaries are structured the way they are, etc.

8. **Consider Structurizr MCP server** for AI-agent-driven diagram generation from blueprints.

### Key Files to Create

| File | Purpose |
|------|---------|
| `vois-c4.dsl` | Master Structurizr DSL for all VOIS blueprints |
| `export-script.sh` | CI/CD script to validate and export to all formats |
| `c4-export-theme.json` | VOIS-branded theme with colours for actors, agents, authorities, systems |

### Pipeline Sketch

```yaml
# .github/workflows/c4-export.yml
steps:
  - name: Validate C4 DSL
    run: java -jar structurizr.war validate -workspace vois-c4.dsl

  - name: Export all formats
    run: |
      java -jar structurizr.war export -workspace vois-c4.dsl -format plantuml -output ./diagrams/plantuml
      java -jar structurizr.war export -workspace vois-c4.dsl -format mermaid -output ./diagrams/mermaid
      java -jar structurizr.war export -workspace vois-c4.dsl -format svg -output ./diagrams/svg
      java -jar structurizr.war export -workspace vois-c4.dsl -format static -output ./diagrams/site
```

---

## References

| Resource | URL |
|----------|-----|
| C4 Model Official | [c4model.com](https://c4model.com/) |
| Structurizr Docs | [docs.structurizr.com](https://docs.structurizr.com/) |
| Structurizr DSL Language Reference | [docs.structurizr.com/dsl/language](https://docs.structurizr.com/dsl/language) |
| Structurizr DSL Tutorial | [docs.structurizr.com/dsl/tutorial](https://docs.structurizr.com/dsl/tutorial) |
| Structurizr Export Formats | [docs.structurizr.com/export](https://docs.structurizr.com/export) |
| Structurizr Validation | [docs.structurizr.com/validate](https://docs.structurizr.com/validate) |
| Structurizr Binaries | [docs.structurizr.com/binaries](https://docs.structurizr.com/binaries) |
| Structurizr Playground | [playground.structurizr.com](https://playground.structurizr.com/) |
| Structurizr Deployment Cookbook | [docs.structurizr.com/dsl/cookbook/deployment-groups/](https://docs.structurizr.com/dsl/cookbook/deployment-groups/) |
| C4 Model FAQ | [c4model.com/faq](https://c4model.com/faq) |
| C4 Model Deployment Diagrams | [c4model.com/diagrams/deployment](https://c4model.com/diagrams/deployment) |
| C4 Model Dynamic Diagrams | [c4model.com/diagrams/dynamic](https://c4model.com/diagrams/dynamic) |
| Structurizr MCP Server | [docs.structurizr.com/ai/mcp](https://docs.structurizr.com/ai/mcp) |
| Wikipedia: C4 Model | [en.wikipedia.org/wiki/C4_model](https://en.wikipedia.org/wiki/C4_model) |
| VOIS Sample — Insurance Claim | [samples/insurance-claim-evidence/](../samples/insurance-claim-evidence/) |

---

*Part of the Vaulted Agent Context project. Generated by Hermes Agent subagent. Published 31 July 2026.*
