# Mosaic Architecture

Architecture documentation for the Mosaic personal intelligence platform.

## Core idea

Mosaic is a local-first personal intelligence layer that runs primarily on the user's home computer. It indexes and connects information from multiple life domains, maintains a unified memory and data model, and exposes cited, permission-aware answers and insights.

Domain applications remain operational clients. They own their local workflows and synchronize selected immutable events through Firebase Authentication and Cloud Firestore. Mosaic Core validates and stores accepted events locally, then builds projections, provenance and cross-domain intelligence.

Mosaic Inventory is a domain module inside `mosaic-android`. It owns household products, ingredients, stock quantities, purchases, expiry dates and stock adjustments. Meal records may reference Inventory items, but Inventory remains the owner of stock deduction decisions.

## Documents

- [System Architecture](SYSTEM_ARCHITECTURE.md)
- [Firebase Synchronization Decision](FIREBASE_SYNC.md)
- [Repository Structure](REPOSITORY_STRUCTURE.md)
- [Data and Memory Model](DATA_AND_MEMORY_MODEL.md)
- [Deployment Topology](DEPLOYMENT.md)
- [API and Event Contracts](API_AND_EVENTS.md)
- [Security and Permissions](SECURITY.md)
- [Phase 1 Plan](PHASE_1.md)
- [Architecture decisions](adrs/)
