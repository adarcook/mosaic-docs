# Mosaic Documentation

System-wide architecture, decisions, roadmap, and operating principles for Mosaic.

## What is Mosaic?

Mosaic is a personal, local-first intelligence ecosystem. Domain applications capture structured information, Mosaic Core builds durable memory and cross-domain insight on the Windows 11 home computer, and the VPS is introduced where always-on remote availability provides concrete value.

## Repositories

- `mosaic-core` — local ingestion, durable memory, indexing, retrieval, permissions, and cross-domain intelligence
- `mosaic-android` — Android domain application for nutrition, Inventory, fitness, swimming, weight, and progress
- `mosaic-server` — always-on VPS APIs, relay, remote jobs, and synchronization where needed
- `mosaic-contracts` — versioned shared schemas and canonical examples
- `mosaic-docs` — this repository

## Core principles

1. Local-first for durable personal intelligence.
2. The VPS provides availability, not unlimited trust.
3. Domain apps own user workflows; Mosaic Core owns cross-domain understanding.
4. Every automated inference should preserve assumptions and confidence.
5. Human confirmation is required before uncertain estimates become trusted records.
6. Components communicate through explicit, versioned contracts.
7. Infrastructure is added only when required by the current vertical slice.

## Documents

- [`ROADMAP.md`](ROADMAP.md) — staged implementation plan, current focus, completion gates, and repo ownership
- [`docs/architecture/README.md`](docs/architecture/README.md) — architecture document index
- [`docs/architecture/PHASE_1.md`](docs/architecture/PHASE_1.md) — detailed first vertical slice
- [`docs/architecture/SYSTEM_ARCHITECTURE.md`](docs/architecture/SYSTEM_ARCHITECTURE.md) — system responsibilities and data flow
- [`docs/architecture/REPOSITORY_STRUCTURE.md`](docs/architecture/REPOSITORY_STRUCTURE.md) — multi-repository boundaries
