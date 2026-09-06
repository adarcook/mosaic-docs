# Phase 1 Plan

## Goal

Deliver one end-to-end nutrition slice that proves Mosaic can:

1. record and correct trusted meal data on Android while fully offline;
2. calculate immediate daily nutrition totals and remaining goals locally;
3. synchronize confirmed immutable meal events asynchronously through Firebase;
4. validate and store accepted events durably in Core with ownership, provenance and correction history;
5. generate an evidence-backed weekly nutrition Insight when the Windows home computer is available;
6. deliver that durable Insight back to Android without depending on FCM for correctness.

Nutrition remains the first slice because meal records are structured, relevant and easy to review manually. The meal model must remain ready for a future Inventory module inside `mosaic-android` without making Inventory a prerequisite for nutrition tracking.

## Deployment assumptions

- Mosaic Core runs directly on the Windows 11 home computer.
- Docker is not required.
- `mosaic-contracts` is consumed by Core as a Git submodule pinned to a reviewed commit.
- Mosaic Android stores operational records locally in Room.
- Android meal recording, correction, daily totals and remaining-goal calculations work without Core or Firebase.
- Firebase Authentication provides cloud identity.
- Cloud Firestore acts as an immutable-event relay while Core may be offline.
- Core remains the durable accepted-event store and asynchronous intelligence layer.
- The home computer is expected to be unavailable for long periods and may primarily run catch-up/deep analysis periodically.
- The VPS is not required for the first synchronization path.
- Photo-assisted meal analysis is optional and should use a replaceable on-device adapter on capable Android hardware rather than requiring the home Core.

## Phase 1 data path

```text
Android manual or confirmed meal
        ↓
Room operational record
        ↓
local deterministic totals and goal progress
        ↓
immutable contract event in local outbox
        ↓
Firebase Auth + Firestore user event path
        ↓
Core Firebase consumer when the home computer is available
        ↓
local contract validation
        ↓
idempotent local event store
        ↓
meal projection and revision history
        ↓
scheduled evidence-backed analysis
        ↓
Firestore Insight
        ↓
Android local Insight inbox
```

Optional meal-analysis assistance happens before the confirmed Room record:

```text
Camera → MealAnalyzer → estimate → user review/correction → confirmed meal
```

The estimate is not trusted business data until confirmation.

## Scope

1. Define versioned meal, event, synchronization, Insight and Inventory-consumption contracts.
2. Make Core installable and testable on Windows 11.
3. Load and validate schemas from the local contracts submodule.
4. Provide a useful offline Android nutrition workflow before synchronization exists.
5. Record and correct meals locally in Room using stable meal and component IDs.
6. Represent meals as structured components with quantity, unit and preparation state when known.
7. Allow optional references from meal components to Inventory items.
8. Calculate daily calories, protein and remaining configured goals deterministically on Android.
9. Establish a Firebase development project and user identity strategy.
10. Define user-scoped Firestore event paths and Security Rules.
11. Create immutable contract events and keep them in a retry-safe Android outbox.
12. Publish events to the authenticated user's Firestore path.
13. Configure Core to consume events only for explicitly allowed users.
14. Validate every downloaded envelope and typed payload before persistence.
15. Persist accepted events idempotently with Firebase owner, source and device metadata.
16. Build normalized meal and component projections.
17. Preserve revision and correction history.
18. Run scheduled/catch-up analysis from confirmed history.
19. Produce a durable evidence-backed weekly nutrition Insight.
20. Synchronize the Insight to Android independently of notification delivery.

## Firebase boundary

Firebase provides identity and online availability, but does not replace Mosaic's domain contracts, Android operational storage or durable Core intelligence layer.

### Firebase owns

- authenticated cloud identity;
- user-scoped event relay;
- durable Insight delivery path;
- limited device and consumer synchronization metadata;
- temporary availability while Android or Core is offline.

### Firebase does not own

- Android operational meal records;
- canonical Core projections;
- long-term personal memory;
- vector or retrieval indexes;
- cross-domain reasoning;
- final contract validation;
- the only copy of accepted data;
- meal photos or unconfirmed analysis estimates by default.

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

The foundation from `mosaic-core` PR #1 is merged into `mosaic-core/main`. A clean run of the documented setup and validation flow on the Windows 11 home computer is still required before this milestone is marked complete.

### 3. Android offline nutrition experience — 🟡 partial / open PRs

Already present in `mosaic-android/main` before the current work:

- Room-backed meal persistence using a temporary `MealAnalysis`-centric compatibility model;
- daily nutrition totals from local data;
- saved-meal history display;
- photo-first analysis/review flow;
- training and other feature-module foundations.

Current open work:

- PR #18 adds locally persisted calorie/protein goals, deterministic remaining-goal calculations, progress UI and unit tests;
- stacked PR #19 adds fully offline manual meal entry, introduces the replaceable `MealAnalyzer` boundary, moves the existing HTTP analyzer behind a legacy adapter, and adds Fit unit tests to Android CI;
- PR #19 CI passed for Fit unit tests, database/photos compilation and `:app:assembleDebug`;
- the stacked branch was installed and smoke-tested on a physical Android device; the new manual capture UI is visible and usable.

Still required before this milestone can be considered complete:

- merge the reviewed Android PRs into `main`;
- edit and delete/correct already-saved meals offline;
- migrate the compatibility storage model to canonical `MealRecord` semantics with stable meal/component IDs and revisions;
- verify restart persistence and accurate totals after corrections;
- keep the completion gate model-independent.

### 4. Firebase identity and security foundation — ⬜ not started

- development Firebase project;
- initial Auth provider;
- Firestore event, device, Insight and consumer paths;
- Security Rules;
- emulator-based ownership and isolation tests;
- configuration and secrets excluded from Git;
- initial retention assumptions documented.

### 5. Android event outbox and Firebase publishing — ⬜ not started

- contract DTO mapping and serialization from canonical Room records;
- immutable event generation;
- retry-safe local outbox;
- Firebase Auth integration;
- authenticated Firestore writes;
- user-scoped event documents;
- upload acknowledgement and retry handling;
- pending, sent and failed states;
- offline and reconnection tests.

### 6. Core Firebase consumer and validation — ⬜ not started

- secure credential loading;
- configured Firebase-user mapping;
- polling or listener consumption;
- envelope and payload validation using pinned schemas;
- malformed and unsupported-event diagnostics;
- efficient checkpoint metadata without relying on it for correctness.

### 7. Durable local event store — ⬜ not started

- initial local database and migrations;
- unique event ID enforcement;
- owner, source and device metadata;
- transaction-safe writes;
- duplicate-safe reprocessing;
- restart, recovery and backup tests.

### 8. Meal projection and history — ⬜ not started

- meal and component projections;
- user-scoped revision ordering;
- correction history;
- current-state and date-range queries;
- provenance from projections to source events.

### 9. Weekly evidence-backed Insight — ⬜ not started

- scheduled and startup catch-up analysis;
- one initial weekly nutrition summary;
- evidence references to exact meal revisions;
- deterministic Insight identity or equivalent deduplication;
- durable Core-side analysis-run history;
- publication to the correct Firestore user path.

### 10. Android Insight inbox and notification signal — ⬜ not started

- Android Insight synchronization and local Room storage;
- read/dismiss/archive state;
- evidence display;
- FCM as a lightweight optional signal only;
- correct behavior when the notification is delayed or missing.

## Meal-analysis assistance boundary

Photo/text analysis is deliberately separated from the trusted meal-record path.

The Android capture UI should depend on a replaceable `MealAnalyzer` interface. On capable devices the preferred production implementation is on-device inference. A remote/HTTP implementation may remain useful for development or an explicitly selected fallback.

The model/runtime is not selected in Phase 1. Selection should follow measurement on representative target hardware, including:

- meal-recognition accuracy;
- nutrition-estimation usefulness;
- memory footprint;
- end-to-end latency;
- battery cost;
- thermal behavior;
- model size and update strategy.

Regardless of runtime, model output remains an estimate until the user confirms or corrects it. Only the resulting confirmed `MealRecord` becomes trusted history and enters the event outbox.

## Acceptance criteria

Phase 1 is complete when all of the following are true:

1. A clean Windows 11 installation can set up and run Core using the provided PowerShell scripts.
2. Core loads pinned contracts locally without runtime GitHub access.
3. A user can record and correct meals on Android while offline, restart the application and immediately see accurate daily calorie/protein totals and remaining configured goals.
4. Android stores meals using canonical stable meal/component IDs and revision semantics.
5. Firebase Security Rules tests prove that users cannot access another user's event or Insight path.
6. A confirmed meal produces a contract-valid immutable event.
7. The event uploads to the authenticated user's Firestore path after connectivity is available.
8. Core consumes only events for an explicitly configured user.
9. Core validates the event against pinned contracts and rejects malformed or unsupported input.
10. The same event may be consumed or delivered more than once without local duplication.
11. The accepted event, owner identity and source metadata survive a Core restart.
12. A correction creates a new revision without erasing the earlier event.
13. Core exposes the latest meal projection while retaining complete correction history.
14. Core can run the due weekly analysis after being offline and produce one evidence-backed Insight without duplicate generation.
15. The Insight appears durably in the correct Android inbox even if FCM delivery fails.
16. Meal components remain valid without Inventory but may optionally reference Inventory items.
17. No uncertain raw-to-cooked conversion silently changes stock.
18. No model estimate becomes a trusted meal record without confirmation.

## Deferred

The following remain outside the Phase 1 completion gate:

- complete Inventory UI and automatic stock deduction;
- recipe-level stock conversion;
- making photo-based meal analysis the only or mandatory input flow;
- selecting or bundling the final on-device meal-analysis model/runtime before representative hardware benchmarking;
- mandatory remote meal analysis or a requirement that the home Core be online during capture;
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

The staged implementation order is maintained in [`../../ROADMAP.md`](../../ROADMAP.md), the Firebase boundary is documented in [`FIREBASE_SYNC.md`](FIREBASE_SYNC.md), and the immediate/asynchronous split is documented in [`PASSIVE_INTELLIGENCE.md`](PASSIVE_INTELLIGENCE.md).
