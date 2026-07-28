# Mosaic Implementation Roadmap

This roadmap defines the order in which Mosaic will be built. Each stage has a concrete outcome and a completion gate.

## Status legend

- ✅ **Complete** — the required work has been merged into the relevant repository's `main` branch.
- 🟡 **In review / verification** — implementation exists in an open PR or still requires local verification.
- ⬜ **Not started** — no completed implementation has been merged into `main`.

## Current status

| Stage | Status | Notes |
|---|---|---|
| 0. Architecture and contracts | ✅ Complete | Architecture documentation and the initial Phase 1 contracts are merged into `main`. |
| 1. Windows Core foundation | 🟡 In review / verification | Implemented in `mosaic-core` PR #1; awaiting Windows 11 verification and merge. |
| 2. Local Core ingestion API | ⬜ Not started | Begins after Stage 1 is verified and merged. |
| 3. Durable local event storage | ⬜ Not started | Depends on the ingestion API. |
| 4. Meal projection and correction history | ⬜ Not started | Depends on durable event storage. |
| 5. Android meal recording and local queue | ⬜ Not started | Depends on stable contracts; may overlap late Stage 4 work. |
| 6. Android-to-Core local synchronization | ⬜ Not started | Depends on Stages 2–5. |
| 7. Retrieval, evidence and cited answers | ⬜ Not started | Completes the first useful end-to-end slice. |
| 8. Meal analysis assistance | ⬜ Not started | Added only after manual meal recording is reliable. |
| 9. Inventory module inside Android | ⬜ Not started | Uses the integration boundary established by the meal contracts. |
| 10. VPS synchronization and remote availability | ⬜ Not started | Added only when away-from-home availability has concrete value. |
| 11. Fitness and swimming domains | ⬜ Not started | Reuses the proven ingestion and evidence pattern. |
| 12. Broader personal intelligence | ⬜ Not started | Photos, documents, projects and controlled automations. |

## Current focus

> Verify the Windows 11 Core foundation locally, then merge it into `main`.

---

## Stage 0 — Architecture and contracts — ✅ Complete

### Goal

Establish stable repository boundaries and a shared language for data exchanged between components.

### Repositories

- `mosaic-docs`
- `mosaic-contracts`

### Merged deliverables

- system architecture and repository responsibilities;
- multi-repository architecture decision;
- versioned meal, event, sync and Inventory-consumption schemas;
- valid JSON examples and a validation script;
- trust, security and deployment boundaries;
- Phase 1 implementation plan.

### Completion gate

A meal record and sync batch can be represented in implementation-neutral JSON and validated against canonical schemas.

### Completion evidence

- `mosaic-docs` architecture PR #1 merged into `main`;
- `mosaic-contracts` Phase 1 contracts PR #1 merged into `main`.

Future contract changes remain normal versioned follow-up work and do not reopen this initial stage.

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
- tests for valid examples, missing submodule and unsupported versions;
- clear Windows installation instructions.

### Completion gate

On the home computer, a clean clone with submodules can run the setup script, load every schema, validate the canonical examples and pass all tests.

### Current state

Implementation exists in `mosaic-core` PR #1. This stage becomes ✅ **Complete** only after the Windows 11 check succeeds and the PR is merged into `main`.

### Explicitly deferred

- Docker;
- database selection;
- FastAPI sync endpoint;
- automatic Windows startup;
- remote access.

---

## Stage 2 — Local Core ingestion API — ⬜ Not started

### Goal

Allow Core to receive versioned events and reject invalid or unsupported input before business storage.

### Deliverables

- minimal FastAPI application bound initially to `127.0.0.1`;
- health and readiness endpoints;
- `POST /v1/sync/batches`;
- validation of sync batch, event envelope and typed payload;
- per-event accepted, duplicate, rejected and unsupported-version results;
- deterministic error responses and tests.

### Completion gate

The canonical sync example is accepted, malformed input is rejected with useful paths, and resending an event returns `duplicate`.

---

## Stage 3 — Durable local event storage — ⬜ Not started

### Goal

Persist accepted events safely and preserve their original form and provenance across Core restarts.

### Deliverables

- initial local database selected from measured needs; SQLite is acceptable for the first single-user slice;
- event store with unique `eventId` enforcement;
- producer, device, source and received-time metadata;
- transaction-safe batch ingestion;
- migrations, restart tests and a local backup procedure.

### Completion gate

An accepted event survives restart, remains deduplicated and can be retrieved exactly with its source metadata.

---

## Stage 4 — Meal projection and correction history — ⬜ Not started

### Goal

Turn raw meal events into a useful current view while retaining prior revisions.

### Deliverables

- normalized meal and component projections;
- revision ordering and conflict rules;
- current-state queries by meal ID and date range;
- correction history without destructive overwrite;
- provenance links from projection fields to source events;
- optional Inventory references retained as metadata.

### Completion gate

A meal can be created, corrected and queried after restart; the latest revision is returned while earlier revisions remain accessible.

---

## Stage 5 — Android meal recording and local queue — ⬜ Not started

### Goal

Create and edit real meal records in Android and prepare them for reliable synchronization.

### Deliverables

- Room entities for meals, components and queued events;
- manual meal entry and editing;
- structured quantity, unit and preparation state;
- optional Inventory item reference;
- mapping between Room entities and contract DTOs;
- contract-compatible JSON serialization;
- retry-safe local queue and fixture tests.

### Completion gate

A meal can be recorded offline, survive restart, be edited and produce a contract-valid batch with stable IDs and revisions.

---

## Stage 6 — Android-to-Core local synchronization — ⬜ Not started

### Goal

Complete the first end-to-end data path over the home network or a direct local connection.

### Deliverables

- configurable Core address;
- initial local-device trust or pairing mechanism;
- queued batch upload;
- acknowledgement, retry and checkpoint handling;
- duplicate-safe resends;
- visible synchronization status and actionable failures in Android.

### Completion gate

A meal created on Android reaches Core, can be sent twice without duplication, survives restart and supports corrected revisions with retained history.

---

## Stage 7 — Retrieval, evidence and cited answers — ⬜ Not started

### Goal

Make trusted records searchable and answer basic questions with resolvable evidence.

### Deliverables

- deterministic retrieval by date, food and nutrition fields;
- evidence objects pointing to exact records and revisions;
- citation resolver;
- basic question endpoint;
- calculation path for daily nutrition totals;
- distinction between stored facts, calculations and inference.

Semantic embeddings are introduced only where they improve retrieval beyond structured queries.

### Completion gate

Core can answer “How much protein did I record today?” and link the answer to the exact meal records used.

This gate completes the first useful Phase 1 vertical slice.

---

## Stage 8 — Meal analysis assistance — ⬜ Not started

### Goal

Reduce manual entry while preserving user control and the distinction between estimates and confirmed facts.

### Deliverables

- photo- or text-assisted analysis;
- replaceable model adapter;
- `meal-analysis` output kept separate from canonical `MealRecord`;
- review, confirmation and correction flow;
- assumptions and confidence retained;
- only confirmed output becomes a trusted revision.

### Completion gate

A generated estimate can be corrected and converted into a canonical meal record without losing the estimate or its assumptions.

---

## Stage 9 — Inventory module inside Android — ⬜ Not started

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

## Stage 10 — VPS synchronization and remote availability — ⬜ Not started

### Goal

Add the VPS only when remote availability provides concrete value, while Core remains the durable intelligence layer.

### Deliverables

- authentication and device authorization;
- encrypted secrets;
- remote event relay and retry behavior;
- minimal retained data with deletion rules;
- Core catch-up after being offline;
- observability, backups and failure handling.

### Completion gate

Android can record away from home, the VPS retains only required relay state and Core catches up safely without duplicates.

---

## Stage 11 — Fitness and swimming domains — ⬜ Not started

### Goal

Apply the proven contracts, synchronization, provenance and retrieval pattern to training data.

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

## Stage 12 — Broader personal intelligence — ⬜ Not started

### Goal

Expand Mosaic beyond health after ingestion, memory, permissions and evidence systems are proven.

### Candidate domains

- local photo search and face/place/object metadata;
- documents, email, calendar and notes;
- software projects and repositories;
- controlled tools, scheduled workflows and personal automations.

Every new domain must define ownership, contracts, permissions, retention, provenance, correction and deletion behavior.

---

## Immediate implementation sequence

```text
✅ 0. Architecture and initial contracts
🟡 1. Verify and merge Windows Core foundation
⬜ 2. Add local ingestion API
⬜ 3. Persist events idempotently
⬜ 4. Build meal projection and history
⬜ 5. Build Android meal recording
⬜ 6. Complete Android-to-Core synchronization
⬜ 7. Add cited retrieval
```

## Working rule

The project has one primary implementation milestone at a time. A stage is marked ✅ **Complete** only when its completion gate is verified and the relevant implementation is merged into `main`.