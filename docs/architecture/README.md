# Mosaic Architecture

Architecture documentation for the Mosaic personal intelligence platform.

## Core idea

Mosaic is a local-first personal intelligence layer that runs primarily on the user's home computer. It indexes and connects information from multiple life domains, maintains a unified memory and data model, and exposes cited, permission-aware answers and insights.

Domain applications such as Mosaic Fit, Mosaic Inventory, Mosaic Swim and Mosaic Photos remain independent domain clients or services. They own their operational workflows and synchronize selected events and records with Mosaic Core.

Mosaic Inventory is responsible for household products, ingredients, stock quantities, purchases, expiry dates and stock adjustments. Meal records from Mosaic Fit may reference Inventory items, but Inventory remains the owner of stock deduction decisions.

## Documents

- [System Architecture](SYSTEM_ARCHITECTURE.md)
- [Repository Structure](REPOSITORY_STRUCTURE.md)
- [Data and Memory Model](DATA_AND_MEMORY_MODEL.md)
- [Deployment Topology](DEPLOYMENT.md)
- [API and Event Contracts](API_AND_EVENTS.md)
- [Security and Permissions](SECURITY.md)
- [Phase 1 Plan](PHASE_1.md)
- [Architecture decisions](adrs/)