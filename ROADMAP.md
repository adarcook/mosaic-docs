# Mosaic Implementation Roadmap

This roadmap defines the order in which Mosaic will be built. Each stage has a concrete outcome and a completion gate.

## Status legend

- ✅ **Complete** — the required work has been verified and merged into the relevant repository's `main` branch.
- 🟡 **In review / verification** — implementation exists in an open PR or still requires local verification.
- ⬜ **Not started** — no completed implementation has been merged into `main`.

## Current architecture direction

```text
Mosaic Android
  ├── Room and local outbox
  └── Firebase Auth + Firestore
                  ↓
          immutable event relay
                  ↓
Mosaic Core on Windows 11
  ├── contract validation
  ├── durable local event store
  ├── projections and history
  └── retrieval, provenance and intelligence
```

Firebase provides identity and synchronization availability. It does not replace local Core storage, `mosaic-contracts`, or Mosaic's durable intelligence layer.

See [`docs/architecture/FIREBASE_SYNC.md`](docs/architecture/FIREBASE_SYNC.md) for the full decision and boundaries.

## Current status

| Stage | Status | Notes |
|---|---|---|
| 0. Architecture and initial contracts | ✅ Complete | Initial architecture and Phase 1 contracts are merged into `main`. |
| 1. Windows Core foundation | 🟡 In review / verification | Implemented in `mosaic-core` PR #1; awaiting Windows 11 verification and merge. |
| 2. Firebase identity and relay foundation | ⬜ Not started | Firebase project, Auth choice, Firestore model, Rules and emulator tests. |
| 3. Android meal recording and event outbox | ⬜ Not started | Room models, manual meals, immutable events and offline queue. |
| 4. Android-to-Firebase publishing | ⬜ Not started | Authenticated user-scoped event upload and sync status. |
| 5. Core Firebase consumer and contract validation | ⬜ Not started | Core reads configured users and validates downloaded events locally. |
| 6. Durable local event storage | ⬜ Not started | Idempotent local event store, ownership, provenance and recovery. |
| 7. Meal projection and correction history | ⬜ Not started | Current meal view with complete revision history. |
| 8. Retrieval, evidence and cited answers | ⬜ Not started | Completes the first useful end-to-end slice. |
| 9. Meal analysis assistance | ⬜ Not started | Added only after manual recording and synchronization are reliable. |
| 10. Inventory module inside Android | ⬜ Not started | Uses the integration boundary established by meal contracts. |
| 11. Fitness and swimming domains | ⬜ Not started | Reuses the proven event, provenance and retrieval pattern. |
| 12. Optional VPS services | ⬜ Not started | Added for inference, integrations or workflows Firebase does not cover. |
| 13. Broader personal intelligence | ⬜ Not started | Photos, documents, projects and controlled automations. |

## Current focus

> Verify the Windows 11 Core foundation locally, merge it into `main`, then establish the Firebase development environment and security boundary.

---

## Stage 0 — Architecture and initial contracts — ✅ Complete

### Goal

Establish repository boundaries and a shared versioned language for data exchanged between components.

### Repositories

- `mosaic-docs`
- `mosaic-contracts`

### Merged deliverables

- system architecture and repository responsibilities;
- multi-repository architecture decision;
- canonical meal, event, sync and Inventory-consumption schemas;
- valid JSON examples and contract validation script;
- initial trust and deployment boundaries;
- Phase 1 implementation plan.

### Completion gate

A meal record and event batch can be represented in implementation-neutral JSON and validated against canonical schemas.

### Completion evidence

- `mosaic-docs` architecture PR #1 merged into `main`;
- `mosaic-contracts` Phase 1 contracts PR #1 merged into `main`.

---

## Stage 1 — Windows Core foundation — 🟡 In review / verification

### Goal

Run the first reliable Mosaic Core foundation directly on Windows 11 without Docker.

### Repository

- `mosaic-core`

### Deliverables

- `mosaic-contracts` as a Git submodule pinned to a reviewed commit;
- PowerShell setup, validation and contract-update scripts;
- Python virtual environment and package configuration;
- schema loader and `$ref` registry;
- event-type and event-version schema registry;
- contract tests and clear Windows installation instructions.

### Completion gate

On the home computer, a clean clone with submodules can run the setup script, load every schema, validate the canonical examples and pass all tests.

### Current state

Implementation exists in `mosaic-core` PR #1. This stage becomes ✅ **Complete** only after the Windows 11 check succeeds and the PR is merged into `main`.

---

## Stage 2 — Firebase identity and relay foundation — ⬜ Not started

### Goal

Create a secure, testable cloud rendezvous point so Android and Core do not need to be online or directly reachable at the same time.

### Repositories and services

- Firebase project configuration
- `mosaic-android`
- `mosaic-core`
- `mosaic-docs`

### Deliverables

- separate development Firebase project;
- initial sign-in provider and user identity strategy;
- user-scoped Firestore event paths;
- device and Core-consumer metadata model;
- Firestore Security Rules;
- Rules tests using the Firebase Emulator Suite;
- secret and configuration handling outside Git;
- documented retention and deletion assumptions for the first slice.

### Completion gate

Two test users are isolated by Rules, an authenticated client can write only to its own event path, and unauthenticated or cross-user access is rejected in emulator tests.

---

## Stage 3 — Android meal recording and event outbox — ⬜ Not started

### Goal

Create and edit real meal records offline and convert them into retry-safe immutable contract events.

### Repository

- `mosaic-android`

### Deliverables

- Room entities for meals, components and queued events;
- manual meal creation and editing;
- stable aggregate IDs and revisions;
- structured quantity, unit and preparation state;
- optional Inventory item references;
- mapping between Room entities and contract DTOs;
- immutable event creation;
- local outbox with pending, sent and failed states;
- fixture tests compatible with `mosaic-contracts`.

### Completion gate

A meal can be recorded and corrected offline, survive restart, and produce contract-valid immutable events with stable IDs and increasing revisions.

---

## Stage 4 — Android-to-Firebase publishing — ⬜ Not started

### Goal

Publish queued Android events to the authenticated user's Firestore event space without losing offline capability.

### Repository

- `mosaic-android`

### Deliverables

- Firebase Authentication integration;
- Firestore client integration;
- writes to `users/{uid}/events/{eventId}` or the final approved equivalent;
- duplicate-safe document IDs based on `eventId`;
- retry and acknowledgement handling;
- user-visible synchronization status;
- tests for sign-out, offline use, reconnection and permission failures.

### Completion gate

A meal event created offline is uploaded after connectivity returns, is stored only under the authenticated user, and a repeated upload does not create a second cloud event.

---

## Stage 5 — Core Firebase consumer and contract validation — ⬜ Not started

### Goal

Allow the Windows Core installation to consume cloud events for explicitly configured users and validate them before business storage.

### Repository

- `mosaic-core`

### Deliverables

- secure Firebase credential and configuration loading;
- explicit mapping of allowed Firebase users to local Core users;
- polling or listener-based event consumption;
- envelope and typed-payload validation using the pinned submodule;
- unsupported-version and malformed-event handling;
- consumer checkpoint metadata for efficiency;
- dead-letter or diagnostic handling for rejected events;
- no reliance on Firestore client Rules as the only Core authorization control.

### Completion gate

Core downloads a canonical event for an allowed user, validates it successfully, rejects malformed or unsupported events, and ignores events for unconfigured users.

---

## Stage 6 — Durable local event storage — ⬜ Not started

### Goal

Persist accepted events safely and preserve their exact original form, ownership and provenance across restarts.

### Repository

- `mosaic-core`

### Deliverables

- initial local database; SQLite is acceptable for the first single-machine slice;
- event store with unique `eventId` enforcement;
- Firebase owner UID, producer device, source and received-time metadata;
- transaction-safe ingestion;
- idempotent processing independent of checkpoints;
- migrations, restart tests and local backup procedure.

### Completion gate

An accepted Firebase event survives restart, remains deduplicated after repeated consumption, and can be retrieved exactly with user and source metadata.

---

## Stage 7 — Meal projection and correction history — ⬜ Not started

### Goal

Turn accepted meal events into a useful current view while retaining every prior revision.

### Repository

- `mosaic-core`

### Deliverables

- normalized meal and component projections;
- revision ordering and conflict rules;
- user-scoped queries by meal ID and date range;
- correction history without destructive overwrite;
- provenance links from projection fields to source events;
- optional Inventory references retained as metadata.

### Completion gate

A meal can be created, corrected and queried after restart; the latest valid revision is returned while earlier revisions remain accessible and attributable to the correct user.

---

## Stage 8 — Retrieval, evidence and cited answers — ⬜ Not started

### Goal

Make trusted records searchable and answer basic questions with resolvable evidence.

### Repository

- `mosaic-core`

### Deliverables

- deterministic retrieval by user, date, food and nutrition fields;
- evidence objects pointing to exact records and revisions;
- citation resolver;
- basic local question endpoint or interface;
- calculation path for daily nutrition totals;
- distinction between stored facts, calculations and inference.

Semantic embeddings are introduced only where they improve retrieval beyond structured queries.

### Completion gate

Core can answer “How much protein did I record today?” for the selected user and link the answer to the exact meal revisions used.

This gate completes the first useful Phase 1 vertical slice.

---

## Stage 9 — Meal analysis assistance — ⬜ Not started

### Goal

Reduce manual entry while preserving user control and the distinction between estimates and confirmed facts.

### Deliverables

- photo- or text-assisted analysis;
- replaceable model adapter, local or remote;
- `meal-analysis` output kept separate from canonical `MealRecord`;
- review, confirmation and correction flow;
- assumptions and confidence retained;
- only confirmed output becomes a trusted meal revision.

### Completion gate

A generated estimate can be corrected and converted into a canonical meal record without losing the original estimate or assumptions.

---

## Stage 10 — Inventory module inside Android — ⬜ Not started

### Goal

Add household stock management as a first-class domain module within `mosaic-android`.

### Deliverables

- products, stock, purchases, expiry dates and adjustments;
- Inventory screens and Room storage;
- optional meal-to-stock links;
- explicit raw-to-cooked conversion records with confidence and confirmation;
- processing of `inventory.consumption.requested`;
- Inventory remains the owner of final deductions.

### Completion gate

A confirmed meal may request consumption from a linked item, uncertain conversions require confirmation and stock history remains auditable.

---

## Stage 11 — Fitness and swimming domains — ⬜ Not started

### Goal

Apply the proven contracts, Firebase relay, local event storage, provenance and retrieval pattern to training data.

### Deliverables

- body measurements and weight;
- strength workouts;
- Wear OS and swimming-session ingestion;
- pace, consistency, decay and weekly-load projections;
- combined nutrition and training summaries;
- cited cross-domain questions.

### Completion gate

Core can answer questions combining confirmed meals, measurements and training sessions with evidence for every source.

---

## Stage 12 — Optional VPS services — ⬜ Not started

### Goal

Add VPS services only when a capability is not appropriately provided by Android, Firebase or the local Core.

### Candidate responsibilities

- remote model inference;
- external webhooks and integrations;
- scheduled cloud workflows;
- media processing too expensive for the phone;
- controlled remote commands;
- vendor-independent relay if Firebase is later replaced.

The VPS must not silently become the canonical personal-intelligence database.

### Completion gate

A concrete capability requires the VPS, has explicit authentication and retention rules, and can fail without corrupting Core's durable local state.

---

## Stage 13 — Broader personal intelligence — ⬜ Not started

### Goal

Expand Mosaic beyond health after identity, synchronization, storage, permissions and evidence systems are proven.

### Candidate domains

- local photo search and face/place/object metadata;
- documents, email, calendar and notes;
- software projects and repositories;
- controlled tools, scheduled workflows and personal automations.

Every new domain must define ownership, contracts, cloud exposure, permissions, retention, provenance, correction and deletion behavior.

---

## Immediate implementation sequence

```text
✅ 0. Architecture and initial contracts
🟡 1. Verify and merge Windows Core foundation
⬜ 2. Establish Firebase Auth, Firestore model and Security Rules
⬜ 3. Build Android meal recording and local event outbox
⬜ 4. Publish immutable Android events to Firebase
⬜ 5. Consume and validate Firebase events in Core
⬜ 6. Persist events idempotently in Core
⬜ 7. Build meal projection and correction history
⬜ 8. Add retrieval and cited answers
```

## Working rule

The project has one primary implementation milestone at a time. A stage is marked ✅ **Complete** only when its completion gate is verified and the relevant implementation is merged into `main`.
