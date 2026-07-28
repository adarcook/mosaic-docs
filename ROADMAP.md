# Mosaic Implementation Roadmap

This roadmap describes the order in which Mosaic will be built. Each stage has a concrete outcome and a completion gate. Later stages should not begin until the previous stage is usable and verified.

## Current status

Completed:

- repository boundaries and system architecture documented;
- multi-repository structure established;
- Phase 1 meal, sync and Inventory-ready contracts defined in `mosaic-contracts`;
- Windows-first contract foundation prepared in `mosaic-core` as a draft PR.

Current focus:

> Make Mosaic Core installable and testable on the Windows 11 home computer, with `mosaic-contracts` available locally as a pinned Git submodule.

## Stage 0 — Architecture and contracts

### Goal

Establish stable repository boundaries and a shared language for data exchanged between components.

### Repositories

- `mosaic-docs`
- `mosaic-contracts`

### Deliverables

- system architecture and repository responsibilities;
- versioned meal, event, sync and Inventory-consumption schemas;
- valid JSON examples;
- contract validation script;
- architecture decisions and trust boundaries.

### Completion gate

A meal record and sync batch can be represented in implementation-neutral JSON and validated against the canonical schemas.

### Status

Complete for the initial Phase 1 contracts. Future contract changes remain versioned follow-up work.

## Stage 1 — Windows Core foundation

### Goal

Run the first reliable Mosaic Core foundation directly on Windows 11 without Docker.

### Repository

- `mosaic-core`

### Deliverables

- `mosaic-contracts` Git submodule pinned to a reviewed commit;
- PowerShell setup, validation and contract-update scripts;
- Python virtual environment and package configuration;
- schema loader and `$ref` registry;
- event-type and event-version schema registry;
- tests for valid examples, missing submodule and unsupported versions;
- clear Windows installation instructions.

### Completion gate

On the home computer, a clean clone with submodules can run the setup script, load every schema, validate the canonical examples and pass all tests.

### Explicitly deferred

- Docker;
- database selection;
- FastAPI sync endpoint;
- automatic Windows startup;
- remote access.

## Stage 2 — Local Core ingestion API

### Goal

Allow Core to receive versioned events and reject invalid or unsupported input before it reaches business storage.

### Repository

- `mosaic-core`

### Deliverables

- minimal FastAPI application bound to `127.0.0.1`;
- health and readiness endpoints;
- `POST /v1/sync/batches`;
- validation of sync batch, event envelope and typed payload;
- per-event accepted, duplicate, rejected and unsupported-version results;
- deterministic error responses;
- request and validation tests.

### Completion gate

The canonical sync example is accepted, malformed examples are rejected with useful paths, and resending the same event returns `duplicate` rather than creating another record.

## Stage 3 — Durable local event storage

### Goal

Persist accepted events safely and preserve their original form and provenance across Core restarts.

### Repository

- `mosaic-core`

### Deliverables

- choose the initial local database based on measured needs; SQLite is acceptable for the first single-user slice;
- event store with unique `eventId` enforcement;
- source, producer, device and received-time metadata;
- transaction-safe batch ingestion;
- restart and recovery tests;
- migrations and local backup procedure.

### Completion gate

An accepted event survives restart, remains deduplicated, and its original payload and source metadata can be retrieved exactly.

## Stage 4 — Meal projection and correction history

### Goal

Turn raw meal events into a useful current view while retaining all prior revisions.

### Repositories

- `mosaic-core`
- `mosaic-contracts` only when a contract revision is required

### Deliverables

- normalized meal and meal-component projections;
- revision ordering and conflict rules;
- current-state query by meal ID and date range;
- correction history without destructive overwrite;
- provenance links from projection fields to source events;
- Inventory references retained as optional metadata.

### Completion gate

A meal can be created, corrected and queried after restart; Core returns the latest revision while preserving and exposing earlier revisions.

## Stage 5 — Android meal recording and local queue

### Goal

Create and edit real meal records in the Android application and prepare them for reliable synchronization.

### Repository

- `mosaic-android`

### Deliverables

- Room entities for meals, components and queued sync events;
- manual meal entry and editing flow;
- structured quantity, unit and preparation state;
- optional Inventory item reference;
- mapping between Room entities and contract DTOs;
- JSON serialization compatible with `mosaic-contracts`;
- retry-safe local sync queue;
- contract fixture tests shared with Core.

### Completion gate

A user can record a meal offline, restart the application, edit it and produce a contract-valid sync batch containing stable IDs and revisions.

## Stage 6 — Android-to-Core local synchronization

### Goal

Complete the first end-to-end vertical slice over the home network or a direct local connection.

### Repositories

- `mosaic-android`
- `mosaic-core`

### Deliverables

- configurable Core address;
- authenticated local-device pairing or an equivalent initial trust mechanism;
- upload of queued batches;
- acknowledgement and retry handling;
- checkpointing and duplicate-safe resends;
- Android display of synchronization state and actionable failures.

### Completion gate

A meal created on Android reaches Core, can be sent twice without duplication, survives Core restart, and a corrected revision becomes the current projection while preserving history.

## Stage 7 — Retrieval, evidence and cited answers

### Goal

Make trusted records searchable and answer basic questions with resolvable evidence.

### Repository

- `mosaic-core`

### Deliverables

- deterministic meal retrieval by date, food and nutrition fields;
- evidence objects that point to exact records and revisions;
- citation resolver;
- basic question endpoint;
- calculation path for questions such as daily protein totals;
- explicit distinction between stored facts, calculations and inferred content.

Semantic embeddings may be added only where they improve retrieval beyond structured queries.

### Completion gate

Core can answer a question such as “How much protein did I record today?” and link each part of the answer to the exact meal records used.

## Stage 8 — Meal analysis assistance

### Goal

Reduce manual entry while preserving user control and the distinction between model estimates and confirmed facts.

### Repositories

- `mosaic-android`
- `mosaic-server` when remote analysis is needed
- `mosaic-core`

### Deliverables

- photo or text-assisted meal analysis;
- replaceable model adapter;
- `meal-analysis` output kept separate from canonical `MealRecord`;
- confirmation and correction interface;
- assumptions, confidence and unanswered questions preserved;
- only confirmed output becomes a trusted meal revision.

### Completion gate

A model-generated estimate can be reviewed, corrected and converted into a canonical meal record without losing the original estimate or its assumptions.

## Stage 9 — Inventory module inside Android

### Goal

Add household stock management as a first-class domain module within `mosaic-android`.

### Repositories

- `mosaic-android`
- `mosaic-contracts`
- `mosaic-core` for cross-domain history and retrieval

### Deliverables

- products, stock quantities, purchases, expiry dates and adjustments;
- Inventory screens and local Room storage;
- optional linking from meal components to stock items;
- explicit raw-to-cooked conversion records with confidence and confirmation;
- processing of `inventory.consumption.requested`;
- Inventory owns final stock deductions; Fit never mutates Inventory tables directly.

### Completion gate

A confirmed meal may request consumption from a linked product, uncertain conversions require confirmation, and stock history remains auditable.

## Stage 10 — VPS synchronization and remote availability

### Goal

Add the VPS only when remote availability provides concrete value, while keeping Core the durable personal intelligence layer.

### Repository

- `mosaic-server`

### Deliverables

- authentication and device authorization;
- encrypted secrets;
- remote event relay and retry behavior;
- minimal retained data with explicit deletion rules;
- Core catch-up after being offline;
- observability, backups and failure handling.

### Completion gate

Android can record data away from home, the VPS holds only the required relay state, and Core safely catches up without duplicates when it becomes available.

## Stage 11 — Fitness and swimming domains

### Goal

Apply the proven contracts, synchronization, provenance and retrieval pattern to training data.

### Repositories

- `mosaic-android`
- `mosaic-contracts`
- `mosaic-core`

### Deliverables

- body measurements and weight;
- strength workouts;
- Wear OS and swimming-session ingestion;
- pace, consistency, decay and weekly-load projections;
- combined nutrition and training summaries;
- cited cross-domain questions.

### Completion gate

Core can answer questions that combine confirmed meals, measurements and training sessions, with evidence for each source.

## Stage 12 — Broader personal intelligence

### Goal

Expand Mosaic beyond health after the underlying ingestion, memory, permissions and evidence systems are proven.

### Candidate domains

- local photo search and face/place/object metadata;
- documents, email, calendar and notes;
- software projects and repositories;
- controlled tools, scheduled workflows and personal automations.

Every new domain must define:

- ownership and repository placement;
- contracts and versioning;
- source permissions;
- retention policy;
- provenance and citation behavior;
- correction and deletion semantics.

## Working rule

At any time, the project should have one primary implementation milestone. New infrastructure should be added only when required by that milestone.

The immediate sequence is:

```text
1. Verify Windows Core foundation
2. Add local ingestion API
3. Persist events idempotently
4. Build meal projection and history
5. Build Android meal recording
6. Complete Android-to-Core synchronization
7. Add cited retrieval
```
