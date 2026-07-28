# Mosaic Documentation

System-wide architecture, decisions, roadmap, and operating principles for Mosaic.

## What is Mosaic?

Mosaic is a personal, local-first intelligence ecosystem. Domain applications capture structured information, Firebase provides cloud identity and event relay, and Mosaic Core builds durable memory and cross-domain insight on the Windows 11 home computer.

Firebase improves availability and multi-user readiness without replacing the local Core data store. The VPS is introduced only where inference, integrations or remote workflows provide concrete value.

## Repositories

- `mosaic-core` — local event ingestion, durable memory, indexing, retrieval, permissions, and cross-domain intelligence
- `mosaic-android` — Android domain application for nutrition, Inventory, fitness, swimming, weight, and progress
- `mosaic-server` — optional VPS inference, integrations, remote jobs, and services where needed
- `mosaic-contracts` — versioned shared schemas and canonical examples
- `mosaic-docs` — this repository

## Core principles

1. Local-first for durable personal intelligence.
2. Firebase provides identity and synchronization availability, not unlimited trust or the only durable copy.
3. Domain apps own user workflows; Mosaic Core owns cross-domain understanding.
4. Every automated inference should preserve assumptions and confidence.
5. Human confirmation is required before uncertain estimates become trusted records.
6. Components communicate through explicit, versioned contracts.
7. Every synchronized record preserves user ownership and provenance.
8. Infrastructure is added only when required by the current vertical slice.

## Documents

- [`ROADMAP.md`](ROADMAP.md) — staged implementation plan, current focus, completion gates, and repository ownership
- [`ARCHITECTURE.md`](ARCHITECTURE.md) — current system overview and source-of-truth boundaries
- [`docs/architecture/README.md`](docs/architecture/README.md) — architecture document index
- [`docs/architecture/FIREBASE_SYNC.md`](docs/architecture/FIREBASE_SYNC.md) — Firebase identity and event-relay decision
- [`docs/architecture/PHASE_1.md`](docs/architecture/PHASE_1.md) — detailed first vertical slice
- [`docs/architecture/SYSTEM_ARCHITECTURE.md`](docs/architecture/SYSTEM_ARCHITECTURE.md) — system responsibilities and data flow
- [`docs/architecture/REPOSITORY_STRUCTURE.md`](docs/architecture/REPOSITORY_STRUCTURE.md) — multi-repository boundaries
