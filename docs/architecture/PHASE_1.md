# Phase 1 Plan

## Goal

Deliver one end-to-end vertical slice that proves Mosaic can receive a trusted meal record from Android, validate and store it locally in Core, preserve provenance and correction history, retrieve it, and answer a cited question.

The first slice uses nutrition because meal records are already relevant, structured and easy to review manually. The meal model must also remain ready for a future Inventory module inside `mosaic-android` without making Inventory a prerequisite for nutrition tracking.

## Deployment assumption

Phase 1 Core runs directly on the Windows 11 home computer.

- Docker is not required.
- `mosaic-contracts` is consumed as a Git submodule pinned to a reviewed commit.
- Core initially binds to localhost while the foundation is tested.
- Local-network and remote synchronization are introduced deliberately after local ingestion is reliable.

## Scope

1. Define versioned meal, event, sync and Inventory-consumption contracts.
2. Make Core installable and testable on Windows 11.
3. Load and validate schemas from the local contracts submodule.
4. Expose a local sync-batch ingestion endpoint.
5. Validate batch, envelope and typed payload before persistence.
6. Persist accepted events idempotently with source metadata.
7. Build normalized meal and component projections.
8. Preserve revision and correction history.
9. Record or confirm a meal in the Android client.
10. Store the operational Android record locally in Room.
11. Represent meals as structured components with quantity, unit and preparation state when known.
12. Allow optional references from meal components to Inventory items.
13. Synchronize retry-safe event batches from Android to Core.
14. Query meal records through the Core API.
15. Return a basic answer with resolvable citations to exact records and revisions.

## Inventory readiness

Phase 1 does not require a complete Inventory interface or automatic stock deduction. It establishes the boundary required for a later Inventory module inside `mosaic-android`.

The meal contract supports:

- stable meal-component IDs;
- normalized food identity where available;
- quantity and unit;
- preparation state such as raw, cooked or unknown;
- an optional Inventory item reference;
- match confidence and whether a match was suggested or user-confirmed;
- correction history and provenance;
- a versioned `inventory.consumption.requested` event.

Inventory owns stock levels, purchases, expiry dates, conversions and final deduction decisions. The Fit/nutrition domain must not directly modify Inventory storage.

## Implementation milestones

### 1. Contracts — complete for the initial slice

- canonical `MealRecord`;
- structured components and nutrition values;
- event envelope and idempotent sync batch;
- optional Inventory references;
- Inventory consumption-request schema;
- valid examples and schema validation.

### 2. Windows Core foundation — current milestone

- contracts Git submodule;
- PowerShell setup and validation scripts;
- Python package and virtual environment;
- schema loader and resource registry;
- event-type/version registry;
- contract tests and clear missing-submodule errors.

### 3. Local ingestion API

- minimal FastAPI application;
- health and readiness endpoints;
- `POST /v1/sync/batches`;
- batch, envelope and payload validation;
- accepted, rejected, duplicate and unsupported-version results.

### 4. Durable event store

- initial local database and migrations;
- unique event ID enforcement;
- source and device metadata;
- transaction-safe batch writes;
- restart and recovery tests;
- local backup procedure.

### 5. Meal projection and history

- meal and component projections;
- revision ordering;
- correction history;
- current-state and date-range queries;
- provenance from projections to source events.

### 6. Android recording

- Room entities for meals, components and queued events;
- manual create and edit flow;
- contract DTO mapping and serialization;
- retry-safe local queue;
- offline persistence and stable IDs.

### 7. Android-to-Core synchronization

- configurable Core address;
- initial local device trust mechanism;
- upload, acknowledgement and retries;
- checkpointing and duplicate-safe resend;
- visible synchronization state and errors.

### 8. Retrieval and cited answer

- structured meal search;
- evidence objects and citation resolver;
- basic question endpoint;
- calculations such as daily protein totals;
- links to exact meal records and revisions.

## Acceptance criteria

Phase 1 is complete when all of the following are true:

1. A clean Windows 11 installation can set up and run Core using the provided PowerShell scripts.
2. Core loads the pinned contracts locally without runtime GitHub access.
3. A confirmed meal created on Android remains available offline and produces a contract-valid sync event.
4. The event can be synchronized twice without duplication.
5. The event and its source metadata survive a Core restart.
6. A correction creates a new revision without erasing the earlier version.
7. Core returns the latest meal projection and can expose its history.
8. A basic answer, such as a daily protein total, links to the exact records used.
9. Meal components remain valid without Inventory but can optionally reference Inventory items.
10. No uncertain raw-to-cooked conversion silently changes stock.

## Deferred

The following remain outside Phase 1:

- complete Inventory UI and automatic stock deduction;
- recipe-level stock conversion;
- photo-based meal analysis as the primary input flow;
- VPS relay and internet-facing APIs;
- Wear OS ingestion and advanced swimming analysis;
- full photo-library indexing;
- autonomous agents;
- multi-user tenancy;
- complex distributed queues;
- automatic Windows service registration.

The staged implementation order is maintained in [`../../ROADMAP.md`](../../ROADMAP.md).
