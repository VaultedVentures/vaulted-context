# Agent Context — Progress Report

## Completed (as of 30 July 2026)

### Phase 1: Foundation ✅

| # | Card | Status | Deliverable |
|---|------|--------|-------------|
| 1 | Draft public context.md | ✅ Done | [`context.md`](context.md) + [`llms.txt`](llms.txt) — published at [github.com/VaultedVentures/vaulted-context](https://github.com/VaultedVentures/vaulted-context) |
| 2 | Define native VOIS JSON schema | ✅ Done | [`schemas/vois-blueprint.json`](schemas/vois-blueprint.json) — JSON Schema (Draft-07) with all primitive definitions |
| 3 | Define structured Markdown format | ✅ Done | [`formats/vois-blueprint.md`](formats/vois-blueprint.md) — YAML frontmatter + tables + Mermaid slot |
| 4 | Add Mermaid blueprint export | ✅ Done | Embedded Mermaid flowchart in sample blueprint |
| 5 | Create sample industry blueprints | ✅ Done | [`samples/insurance-claim-evidence/`](samples/insurance-claim-evidence/) — full Markdown + JSON blueprint |

### Phase 2: Simulator Integration ✅

| # | Card | Status | Deliverable |
|---|------|--------|-------------|
| 6 | Design Import AI blueprint flow | ✅ Done | [`designs/import-ai-blueprint-flow.md`](designs/import-ai-blueprint-flow.md) — full flow: AI generate → import → validate → review → render |
| 7 | Design homepage context file surfacing | ✅ Done | [`designs/homepage-context-surfacing.md`](designs/homepage-context-surfacing.md) — hero CTA, /context.md route, footer, agent discovery points |

### Phase 3: Interoperability Research 🔄 In Progress

| # | Card | Status |
|---|------|--------|
| 8 | Research BPMN 2.0 export mapping | 🔄 Researching |
| 9 | Research JSON-LD semantic export | 🔄 Researching |
| 10 | Research W3C PROV provenance mapping | 🔄 Researching |
| 11 | Research OSCAL controls export | 🔄 Researching |
| 12 | Research OpenAPI and AsyncAPI exports | 🔄 Researching |
| 13 | Research C4 and Structurizr exports | 🔄 Researching |

## Repo Structure

```
vaulted-context/
├── context.md                     # Canonical AI-readable orientation
├── llms.txt                       # AI crawler entry point
├── README.md                      # Repo overview
├── schemas/
│   └── vois-blueprint.json        # Native JSON Schema
├── formats/
│   └── vois-blueprint.md          # Structured Markdown template
├── designs/
│   ├── import-ai-blueprint-flow.md
│   └── homepage-context-surfacing.md
├── samples/
│   └── insurance-claim-evidence/
│       ├── README.md              # Full blueprint in Markdown
│       └── vois-blueprint.json    # Full blueprint in JSON
└── research/                      # (being populated by subagents)
    ├── bpmn-export.md
    ├── jsonld-export.md
    ├── w3c-prov-mapping.md
    ├── oscal-export.md
    ├── openapi-asyncapi-export.md
    └── c4-structurizr-export.md
```
