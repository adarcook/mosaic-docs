# Phase 1 Plan

## Goal

Deliver one end-to-end vertical slice that proves Mosaic can receive a trusted meal record from Android through Firebase, validate and store it locally in Core, preserve ownership, provenance and correction history, retrieve it, and answer a cited question.

The first slice uses nutrition because meal records are structured, relevant and easy to review manually. The meal model must remain ready for a future Inventory module inside `mosaic-android` without making Inventory a prerequisite for nutrition tracking.

## Deployment assumptions

- Mosaic Core runs directly on the Windows 11 home computer.
- Docker is not required.
- `mosaic-contracts` is consumed by Core as a Git submodule pinned to a reviewed commit.
- Mosaic Android stores operational records locally in Room.
- Firebase Authentication provides cloud identity.
- Cloud Firestore acts as an immutable-event relay while Core may be offline.
- Core remains the durable accepted-event store and intelligence layer.
- The VPS is not required for the first synchronization path.

## Phase 1 data path

```text
Android meal in Room
        ↓
immutable contract event in local outbox
        ↓
Firebase Auth + Firestore user event path
        ↓
Core Firebase consumer
        ↓
local contract validation
        ↓
idempotent local event store
        ↓
meal projection and revision history
        ↓
retrieval and cited answer
```

## Scope

1. Define versioned meal, event, synchronization and Inventory-consumption contracts.
2. Make Core installable and testable on Windows 11.
3. Load and validate schemas from the local contracts submodule.
4. Establish a Firebase development project and user identity strategy.
5. Define user-scoped Firestore event paths and Security Rules.
6. Record or confirm a meal in Android and store it locally in Room.
7. Represent meals as structured components with quantity, unit and preparation state when known.
8. Allow optional references from meal components to Inventory items.
9. Create immutable contract events and keep them in a retry-safe Android outbox.
10. Publish events to the authenticated user's Firestore path.
11. Configure Core to consume events only for explicitly allowed users.
12. Validate every downloaded envelope and typed payload before persistence.
13. Persist accepted events idempotently with Firebase owner, source and device metadata.
14. Build normalized meal and component projections.
15. Preserve revision and correction history.
16. Query meal records locally through Core.
17. Return a basic answer with resolvable citations to exact records and revisions.

## Firebase boundary

Firebase provides identity and online availability, but does not replace Mosaic's domain contracts or durable intelligence layer.

### Firebase owns

- authenticated cloud identity;
- user-scoped event relay;
- limited device and consumer synchronization metadata;
- temporary availability while Android or Core is offline.

### Firebase does not own

- canonical meal projections;
- long-term personal memory;
- vector or retrieval indexes;
- cross-domain reasoning;
- final contract validation;
- the only copy of accepted data.

Corrections are new immutable events. Android and Core must not depend on destructive Firestore updates to preserve business history.

## Multi-user readiness

Phase 1 may be tested with one real user, but it must not hard-code a global user.

Every synchronized event must be associated with:

- Firebase `uid`;
- producer device ID;
- stable event ID;
- aggregate ID and revision where applicable.

Core must map configured Firebase users to local Mosaic users and keep user ownership in the local event store and projections.

Firestore Security Rules must reject unauthenticated and cross-user access. Because server SDK credentials use IAM rather than client Rules, Core must also constrain which users it consumes independently.

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

Inventory owns stock levels, purchases, expiry dates, conversions and final deduction decisions. The nutrition domain must not directly modify Inventory storage.

## Implementation milestones

### 1. Contracts — ✅ complete for the initial slice

- canonical `MealRecord`;
- structured components and nutrition values;
- event envelope and sync batch;
- optional Inventory references;
- Inventory consumption-request schema;
- valid examples and schema validation.

### 2. Windows Core foundation — 🟡 merged; runtime verification pending

- contracts Git submodule;
- PowerShell setup and validation scripts;
- Python package and virtual environment;
- schema loader and resource registry;
- event-type/version registry;
- contract tests and clear missing-submodule errors.

The foundation is merged into `mosaic-core/main`. A clean run of the documented setup and validation flow on the Windows 11 home computer is still required before this milestone is marked complete.

### 3. Firebase identity and security foundation

- development Firebase project;
- initial Auth provider;
- Firestore event, device and consumer paths;
- Security Rules;
- emulator-based ownership and isolation tests;
- configuration and secrets excluded from Git;
- initial retention assumptions documented.

### 4. Android meal recording and outbox

- Room entities for meals, components and queued events;
- manual create and edit flow;
- contract DTO mapping and serialization;
- immutable event generation;
- retry-safe local outbox;
- offline persistence and stable IDs.

### 5. Android Firebase publishing

- Firebase Auth integration;
- authenticated Firestore writes;
- user-scoped event documents;
- upload acknowledgement and retry handling;
- pending, sent and failed states;
- offline and reconnection tests.

### 6. Core Firebase consumer and validation

- secure credential loading;
- configured Firebase-user mapping;
- polling or listener consumption;
- envelope and payload validation using pinned schemas;
- malformed and unsupported-event diagnostics;
- efficient checkpoint metadata without relying on it for correctness.

### 7. Durable local event store

- initial local database and migrations;
- unique event ID enforcement;
- owner, source and device metadata;
- transaction-safe writes;
- duplicate-safe reprocessing;
- restart, recovery and backup tests.

### 8. Meal projection and history

- meal and component projections;
- user-scoped revision ordering;
- correction history;
- current-state and date-range queries;
- provenance from projections to source events.

### 9. Retrieval and cited answer

- structured meal search;
- evidence objects and citation resolver;
- basic local question interface;
- calculations such as daily protein totals;
- links to exact meal records and revisions.

## Acceptance criteria

Phase 1 is complete when all of the following are true:

1. A clean Windows 11 installation can set up and run Core using the provided PowerShell scripts.
2. Core loads pinned contracts locally without runtime GitHub access.
3. Firebase Security Rules tests prove that users cannot access another user's event path.
4. A confirmed meal created on Android remains available offline and produces a contract-valid immutable event.
5. The event uploads to the authenticated user's Firestore path after connectivity is available.
6. Core consumes only events for an explicitly configured user.
7. Core validates the event against pinned contracts and rejects malformed or unsupported input.
8. The same event may be consumed or delivered more than once without local duplication.
9. The accepted event, owner identity and source metadata survive a Core restart.
10. A correction creates a new revision without erasing the earlier event.
11. Core returns the latest meal projection and can expose its complete history.
12. A basic answer, such as a daily protein total, links to the exact records used.
13. Meal components remain valid without Inventory but may optionally reference Inventory items.
14. No uncertain raw-to-cooked conversion silently changes stock.

## Deferred

The following remain outside Phase 1:

- complete Inventory UI and automatic stock deduction;
- recipe-level stock conversion;
- photo-based meal analysis as the primary input flow;
- VPS relay or public Core API;
- production-scale multi-tenant administration;
- household sharing and complex per-field permissions;
- automatic cloud-event deletion before a retention strategy is approved;
- application-level payload encryption unless required by the initial privacy review;
- Wear OS ingestion and advanced swimming analysis;
- full photo-library indexing;
- autonomous agents;
- complex distributed queues;
- automatic Windows service registration.

The staged implementation order is maintained in [`../../ROADMAP.md`](../../ROADMAP.md), and the Firebase boundary is documented in [`FIREBASE_SYNC.md`](FIREBASE_SYNC.md).
