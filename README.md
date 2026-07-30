# Vaulted Ventures — AI Context

This repository provides the canonical AI-readable orientation for **Vaulted Ventures** and the
**Vaulted Objects Infrastructure (VOI)**.

## Contents

| File | Purpose |
|------|---------|
| [`context.md`](context.md) | Full canonical orientation — company, VOI, ITS, FAQ, audience summaries, blueprint guidance |
| [`llms.txt`](llms.txt) | Lightweight AI crawler entry point |
| [`schemas/vois-blueprint.json`](schemas/vois-blueprint.json) | Native JSON Schema for VOIS blueprints (lossless canonical format) |
| [`formats/vois-blueprint.md`](formats/vois-blueprint.md) | Structured Markdown blueprint template with YAML frontmatter and Mermaid diagrams |
| [`samples/insurance-claim-evidence/`](samples/insurance-claim-evidence/) | Full sample blueprint: insurance claim evidence workflow (Markdown + JSON) |

## Quick Start for AI Assistants

1. Read [`context.md`](context.md) first — it contains preferred terminology, primitive
   definitions, and audience-specific summaries
2. Use [`schemas/vois-blueprint.json`](schemas/vois-blueprint.json) for validating or
   generating native blueprints
3. Use [`formats/vois-blueprint.md`](formats/vois-blueprint.md) as a template for
   human-readable blueprint exports
4. See [`samples/insurance-claim-evidence/`](samples/insurance-claim-evidence/) for a
   complete worked example

## Terminology Rules

| Use | Don't Use |
|-----|-----------|
| Information-Theoretic Security (ITS) | Post-Quantum Cryptography (PQC) |
| Zero-Exposure infrastructure | Blockchain, DLT, ledger |
| ITS digital seals / signatures | Quantum-safe signatures |
| Vaulted Objects Infrastructure | Keyless or no-key crypto |

## Public Links

- [vaulted.ventures](https://vaulted.ventures) — company website
- [simulator.vaulted.ventures](https://simulator.vaulted.ventures) — VOI Simulator
- [github.com/VaultedVentures](https://github.com/VaultedVentures) — organisation GitHub

---

*Part of the Agent Context project. Published 30 July 2026.*
