# Mosaic Implementation Roadmap

This roadmap defines the order in which Mosaic will be built. Each stage has a concrete outcome and a completion gate.

## Status legend

- ✅ **Complete** — the required work has been verified and merged into the relevant repository's `main` branch.
- 🟡 **In review / verification** — implementation exists in an open PR or still requires local verification.
- ⬜ **Not started** — no completed implementation has been merged into `main`.

## Product direction

Mosaic is not initially an always-available conversational server.

```text
Immediate local experience
Android Room + deterministic calculations

Asynchronous intelligence
Android events
    ↓
Firebase Auth + Firestore event relay
    ↓
Mosaic Core on Windows 11
    ↓
periodic analysis and durable Insights
    ↓
Firestore Insight inbox
    ↓
FCM notification signal
    ↓
Android displays the Insight
```

Android answers immediate questions that depend only on local structured data, such as today's protein total and the remaining amount toward a configured goal.

Core performs deeper analysis when the home computer is available. Its initial user-facing outputs are weekly summaries, detected patterns and recommendation candidates rather than remote conversational answers.

Firebase provides identity, asynchronous synchronization and notification delivery. It does not replace local Core storage, `mosaic-contracts`, Android Room or the durable Insight records stored before an FCM signal is sent.

See:

- [`docs/architecture/FIREBASE_SYNC.md`](docs/architecture/FIREBASE_SYNC.md)
- [`docs/architecture/PASSIVE_INTELLIGENCE.md`](docs/architecture/PASSIVE_INTELLIGENCE.md)

## Current status

| Stage | Status | Notes |
|---|---|---|
| 0. Architecture and initial contracts | ✅ Complete | Initial architecture and Phase 1 contracts are merged into `main`. |
| 1. Windows Core foundation | 🟡 In review / verification | Implemented in `mosaic-core` PR #1; awaiting Windows 11 verification and merge. |
| 2. Firebase identity and relay foundation | ⬜ Not started | Firebase project, Auth, Firestore paths, Rules and emulator tests. |
| 3. Android meal recording and local dashboard | ⬜ Not started | Room models, manual meals, daily totals and remaining-goal calculations. |
| 4. Android event outbox and Firebase publishing | ⬜ Not started | Immutable contract events, offline queue and authenticated publishing. |
| 5. Core Firebase consumer and contract validation | ⬜ Not started | Core reads configured users and validates downloaded events locally. |
| 6. Durable local event storage | ⬜ Not started | Idempotent event store, ownership, provenance and recovery. |
| 7. Meal projection and correction history | ⬜ Not started | Current meal view with retained revisions and source links. |
| 8. Scheduled analysis and Insight generation | ⬜ Not started | Weekly, catch-up-safe, evidence-backed summaries and patterns. |
| 9. Insight synchronization and Android inbox | ⬜ Not started | Durable Core-to-Android Insight delivery through Firestore. |
| 10. FCM notification delivery | ⬜ Not started | Selective push signals, device registration and user preferences. |
| 11. Meal analysis assistance | ⬜ Not started | Added only after manual recording and the passive loop are reliable. |
| 12. Inventory module inside Android | ⬜ Not started | Uses the integration boundary established by meal contracts. |
| 13. Fitness and swimming domains | ⬜ Not started | Reuses the event, analysis, Insight and notification pattern. |
| 14. Optional VPS services | ⬜ Not started | Added only for capabilities Android, Firebase and Core do not cover. |
| 15. Broader personal intelligence | ⬜ Not started | Photos, documents, projects and controlled automations. |

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
- contract tests and complete Windows installation instructions.

### Completion gate

On the home computer, a clean clone with submodules can run the setup script, load every schema, validate the canonical examples and pass all tests.

### Current state

Implementation exists in `mosaic-core` PR #1. This stage becomes ✅ **Complete** only after the Windows 11 check succeeds and the PR is merged into `main`.

---

## Stage 2 — Firebase identity and relay foundation — ⬜ Not started

### Goal

Create a secure cloud rendezvous point so Android and Core do not need to be online or directly reachable at the same time.

### Repositories and services

- Firebase project configuration
- `mosaic-android`
- `mosaic-core`
- `mosaic-docs`

### Deliverables

- separate development Firebase project;
- initial sign-in provider and stable user identity strategy;
- user-scoped Firestore paths for events, Insights and devices;
- Core-consumer metadata model;
- Firestore Security Rules;
- Rules tests using the Firebase Emulator Suite;
- credentials and configuration kept outside Git;
- initial retention and deletion assumptions.

### Completion gate

Two test users are isolated by Rules, authenticated clients can access only their own paths, and unauthenticated or cross-user access is rejected in emulator tests.

---

## Stage 3 — Android meal recording and local dashboard — ⬜ Not started

### Goal

Provide a useful offline nutrition experience before Core or Firebase analysis is available.

### Repository

- `mosaic-android`

### Deliverables

- Room entities for meals and components;
- manual meal creation, editing and deletion semantics;
- stable meal and component IDs;
- quantity, unit, preparation state and nutrition values;
- optional Inventory item references;
- daily protein and calorie totals;
- configured daily goals;
- deterministic remaining-goal calculation;
- local history by date;
- immediate updates after a meal is recorded or corrected.

### Completion gate

A user can record and correct meals offline, restart the application, and immediately see accurate daily totals and the remaining amount toward the configured protein goal without Core, Firebase or a model call.

---

## Stage 4 — Android event outbox and Firebase publishing — ⬜ Not started

### Goal

Convert confirmed Android records into retry-safe immutable events and publish them to the authenticated user's Firestore event space.

### Repository

- `mosaic-android`

### Deliverables

- mapping between Room records and contract DTOs;
- stable aggregate IDs and increasing revisions;
- immutable meal-created and meal-updated events;
- local outbox with pending, sent and failed states;
- Firebase Authentication integration;
- Firestore event publishing;
- duplicate-safe document IDs based on `eventId`;
- retry, reconnection and permission-failure handling;
- visible synchronization state.

### Completion gate

A meal created offline produces a contract-valid immutable event, uploads after connectivity returns, remains scoped to the authenticated user and does not create a second cloud event when retried.

---

## Stage 5 — Core Firebase consumer and contract validation — ⬜ Not started

### Goal

Allow Core to consume cloud events for explicitly configured users and reject invalid input before business storage.

### Repository

- `mosaic-core`

### Deliverables

- secure Firebase credential and configuration loading;
- mapping of allowed Firebase users to local Core users;
- polling or listener-based event consumption;
- envelope and typed-payload validation using the pinned contracts submodule;
- unsupported-version and malformed-event handling;
- checkpoint metadata for efficiency;
- diagnostic or dead-letter handling;
- no reliance on client Security Rules as the only Core authorization control.

### Completion gate

Core downloads a canonical event for an allowed user, validates it successfully, rejects malformed or unsupported events and ignores events for unconfigured users.

---

## Stage 6 — Durable local event storage — ⬜ Not started

### Goal

Persist accepted events safely and preserve their original form, ownership and provenance across Core restarts.

### Repository

- `mosaic-core`

### Deliverables

- initial local database; SQLite is acceptable for the first single-machine slice;
- unique `eventId` enforcement;
- Firebase UID, producer device, source and received-time metadata;
- transaction-safe ingestion;
- idempotent processing independent of checkpoints;
- migrations, restart tests and backup procedure.

### Completion gate

An accepted Firebase event survives restart, remains deduplicated after repeated consumption and can be retrieved exactly with its ownership and source metadata.

---

## Stage 7 — Meal projection and correction history — ⬜ Not started

### Goal

Turn accepted events into a useful current meal view while retaining every prior revision.

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

A meal can be created, corrected and queried after restart; Core exposes the latest valid revision while all earlier revisions remain accessible and attributable to the correct user.

---

## Stage 8 — Scheduled analysis and Insight generation — ⬜ Not started

### Goal

Turn synchronized history into useful proactive intelligence without requiring the user to ask Core a question.

### Repository

- `mosaic-core`

### Deliverables

- scheduled analysis runner;
- catch-up behavior when the computer was offline;
- per-user analysis checkpoints;
- initial weekly nutrition summary;
- detected-pattern and recommendation-candidate outputs;
- evidence references to exact meals and revisions;
- clear distinction between stored facts, calculations and inference;
- deterministic Insight IDs or equivalent deduplication;
- analysis-run history and failure diagnostics.

### Initial Insight examples

- days in which the protein goal was reached;
- average protein difference between training and rest days;
- repeated evening hunger or late protein concentration;
- missing or incomplete logging periods;
- recommendation candidates that state their evidence and uncertainty.

### Completion gate

After new meal data is synchronized, Core can run a weekly analysis exactly once for the relevant period and produce a durable evidence-backed Insight without user interaction.

This is the first Core intelligence milestone.

---

## Stage 9 — Insight synchronization and Android inbox — ⬜ Not started

### Goal

Deliver Core outputs to Android reliably even if push notifications are delayed or never arrive.

### Repositories

- `mosaic-core`
- `mosaic-android`

### Deliverables

- versioned Insight contract;
- user-scoped Firestore path such as `users/{uid}/insights/{insightId}`;
- durable Insight publication by Core;
- Android Insight synchronization;
- local Room storage for downloaded Insights;
- unread, read, dismissed and archived states;
- evidence and source display;
- duplicate-safe Insight synchronization;
- retry and offline behavior.

### Completion gate

A weekly Insight created by Core appears in the correct user's Android Insight inbox, survives app restart and remains available even when no push notification was received.

This completes the first useful passive end-to-end slice.

---

## Stage 10 — FCM notification delivery — ⬜ Not started

### Goal

Notify the user selectively when a durable Insight is available, without treating notification delivery as the source of truth.

### Repositories and services

- `mosaic-android`
- `mosaic-core` or a trusted Firebase notification dispatcher
- Firebase Cloud Messaging

### Deliverables

- per-installation device registration;
- FCM token refresh and stale-token removal;
- notification preferences by category;
- quiet hours and maximum notification frequency;
- severity and notification-worthiness rules;
- privacy-safe notification payload containing an `insightId`, not the full analysis;
- deep link to the relevant Insight;
- retry and delivery-attempt tracking;
- deduplication so one Insight does not repeatedly notify the same device;
- fallback behavior where Android still discovers the Insight through normal synchronization.

### Completion gate

A durable weekly Insight triggers at most one allowed notification per registered device, respects preferences and quiet hours, opens the correct Insight when tapped, and remains discoverable when FCM delivery fails.

---

## Stage 11 — Meal analysis assistance — ⬜ Not started

### Goal

Reduce manual entry while preserving user control and the distinction between estimates and confirmed facts.

### Deliverables

- photo- or text-assisted meal analysis;
- replaceable local or remote model adapter;
- `meal-analysis` kept separate from canonical `MealRecord`;
- review, confirmation and correction flow;
- retained assumptions and confidence;
- only confirmed output becomes a trusted revision.

### Completion gate

A generated estimate can be corrected and converted into a canonical meal record without losing the original estimate, assumptions or confidence.

---

## Stage 12 — Inventory module inside Android — ⬜ Not started

### Goal

Add household stock management as a first-class domain module within `mosaic-android`.

### Deliverables

- products, stock, purchases, expiry dates and adjustments;
- Inventory screens and Room storage;
- optional meal-to-stock links;
- explicit raw-to-cooked conversions with confidence and confirmation;
- processing of `inventory.consumption.requested`;
- Inventory remains the owner of final deductions;
- Inventory-related passive Insights and optional notifications.

### Completion gate

A confirmed meal may request consumption from a linked item, uncertain conversions require confirmation and stock history remains auditable.

---

## Stage 13 — Fitness and swimming domains — ⬜ Not started

### Goal

Apply the proven contracts, Firebase relay, local storage, scheduled analysis, Insight and notification pattern to training data.

### Deliverables

- body measurements and weight;
- strength workouts;
- Wear OS and swimming-session ingestion;
- pace, consistency, decay and weekly-load projections;
- combined nutrition and training analysis;
- evidence-backed weekly summaries and recommendation candidates;
- selective FCM notifications for useful results.

### Completion gate

Core produces a passive weekly Insight that combines confirmed meals, measurements and training sessions, with evidence for every source, and delivers it to Android.

---

## Stage 14 — Optional VPS services — ⬜ Not started

### Goal

Add VPS services only when a capability is not appropriately provided by Android, Firebase or the local Core.

### Candidate responsibilities

- remote model inference;
- external webhooks and integrations;
- scheduled cloud workflows that must run while Core is offline;
- media processing too expensive for the phone;
- controlled remote commands;
- vendor-independent relay if Firebase is later replaced.

The VPS must not silently become the canonical personal-intelligence database.

### Completion gate

A concrete capability requires the VPS, has explicit authentication and retention rules, and can fail without corrupting Core's durable local state.

---

## Stage 15 — Broader personal intelligence — ⬜ Not started

### Goal

Expand Mosaic beyond health after identity, synchronization, storage, analysis, permissions and evidence systems are proven.

### Candidate domains

- local photo search and face/place/object metadata;
- documents, email, calendar and notes;
- software projects and repositories;
- controlled tools, scheduled workflows and personal automations.

Every new domain must define ownership, contracts, cloud exposure, permissions, retention, provenance, correction, Insight behavior and notification policy.

---

## Immediate implementation sequence

```text
✅ 0. Architecture and initial contracts
🟡 1. Verify and merge Windows Core foundation
⬜ 2. Establish Firebase Auth, Firestore paths and Security Rules
⬜ 3. Build Android meal recording and immediate local calculations
⬜ 4. Build the Android event outbox and Firebase publishing
⬜ 5. Consume and validate Firebase events in Core
⬜ 6. Persist events idempotently in Core
⬜ 7. Build meal projection and correction history
⬜ 8. Generate a weekly evidence-backed Insight
⬜ 9. Synchronize the Insight to the Android inbox
⬜ 10. Signal the Insight selectively through FCM
```

## First useful product milestone

The first product milestone is reached when:

1. Android records meals offline and immediately shows daily protein totals and the remaining amount toward the user's goal.
2. Events synchronize asynchronously through Firebase.
3. Core catches up when the Windows computer becomes available.
4. Core generates one evidence-backed weekly Insight.
5. The Insight appears durably in the Android inbox.
6. An optional FCM notification points to that Insight without being required for delivery correctness.

## Working rule

The project has one primary implementation milestone at a time. A stage is marked ✅ **Complete** only when its completion gate is verified and the relevant implementation is merged into `main`.