# Mosaic Documentation

System-wide architecture, decisions, roadmap, and operating principles for Mosaic.

## What is Mosaic?

Mosaic is a personal, local-first intelligence ecosystem. Domain applications capture structured information, an always-on VPS handles online workflows, and Mosaic Core builds durable memory and cross-domain insight when the home computer is available.

## Repositories

- `mosaic-core` — long-term memory, indexing, retrieval, permissions, and cross-domain intelligence
- `mosaic-fit` — Android application for nutrition, fitness, swimming, weight, and progress
- `mosaic-server` — always-on FastAPI service on the VPS
- `mosaic-contracts` — versioned shared schemas
- `mosaic-docs` — this repository

## Core principles

1. Local-first for durable personal intelligence.
2. The VPS provides availability, not unlimited trust.
3. Domain apps own user workflows; Mosaic Core owns cross-domain understanding.
4. Every automated inference should preserve assumptions and confidence.
5. Human confirmation is required before uncertain nutrition estimates become trusted records.
6. Components communicate through explicit, versioned contracts.

## Documents

- [`ARCHITECTURE.md`](ARCHITECTURE.md)
- [`ROADMAP.md`](ROADMAP.md)
