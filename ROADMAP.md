# Mosaic Implementation Roadmap

This roadmap defines the order in which Mosaic will be built. Each stage has a concrete outcome and a completion gate.

## Status legend

- ✅ **Complete** — the required work has been verified and merged into the relevant repository's `main` branch.
- 🟡 **Partial / verification required** — relevant implementation exists in `main` or an open PR, but the stage completion gate has not been verified and merged end to end.
- ⬜ **Not started** — no implementation that materially advances the stage completion gate has been verified in `main`.

## Product direction

Mosaic is not initially an always-available conversational server.

```text
Immediate local experience
Android Room
  ├── manual capture and correction
  ├── deterministic calculations and goals
  └── optional on-device model assistance

Asynchronous intelligence
Android confirmed events
    ↓
Firebase Auth + Firestore event relay
    ↓
Mosaic Core on Windows 11
    ↓
periodic / catch-up deep analysis and durable Insights
    ↓
Firestore Insight inbox
    ↓
FCM notification signal
    ↓
Android displays the Insight
```

Android owns the immediate operational experience. Meal recording, correction, daily calorie/protein totals and remaining-goal calculations must work while Firebase is unavailable and the home computer is off.

Photo-assisted meal analysis is an optional capture aid. The preferred production direction is a replaceable on-device analyzer on capable Android hardware. Model output remains an estimate until the user confirms or corrects it; only confirmed structured meal data becomes trusted history.

Core performs deeper historical and cross-domain analysis when the home computer is available. Its initial user-facing outputs are weekly summaries, detected patterns and recommendation candidates rather than remote conversational answers or an always-on meal-analysis endpoint.

Firebase provides identity, asynchronous synchronization and notification delivery. It does not replace local Core storage, `mosaic-contracts`, Android Room or the durable Insight records stored before an FCM signal is sent.

See:

- [`docs/architecture/SYSTEM_ARCHITECTURE.md`](docs/architecture/SYSTEM_ARCHITECTURE.md)
- [`docs/architecture/FIREBASE_SYNC.md`](docs/architecture/FIREBASE_SYNC.md)
- [`docs/architecture/PASSIVE_INTELLIGENCE.md`](docs/architecture/PASSIVE_INTELLIGENCE.md)
- [`docs/architecture/API_AND_EVENTS.md`](docs/architecture/API_AND_EVENTS.md)
- [`docs/architecture/SECURITY.md`](docs/architecture/SECURITY.md)

## Current status

_Last re-baselined against the five repositories on 2026-09-06._

Status tracks completion gates, not merely the presence of code. Open PR work remains work in progress even when CI or device smoke testing succeeds.

| Stage | Status | Notes |
|---|---|---|
| 0. Architecture and initial contracts | ✅ Complete | Initial architecture and Phase 1 contracts are merged into `main`. |
| 1. Windows Core foundation | 🟡 Partial / verification required | Foundation from `mosaic-core` PR #1 is merged into `main`; clean Windows 11 verification is still required. |
| 2. Firebase identity and relay foundation | ⬜ Not started | Firebase project, Auth, Firestore paths, Rules and emulator tests are not implemented. |
| 3. Android meal recording and local dashboard | 🟡 Partial / verification required | PRs #18–#20 are merged: local goals/progress, offline manual capture, analyzer boundary and saved-meal edit/delete are in `main`. Draft PR #21 adds canonical UUID meal/component identity plus append-only local revisions; device migration verification is still required. |
| 4. Android event outbox and Firebase publishing | ⬜ Not started | Immutable contract events, offline queue and authenticated publishing are not implemented. |
| 5. Core Firebase consumer and contract validation | ⬜ Not started | Core Firebase consumption is not implemented. |
| 6. Durable local event storage | ⬜ Not started | Idempotent accepted-event storage, ownership and recovery are not implemented. |
| 7. Meal projection and correction history | ⬜ Not started | Canonical event-driven meal projection and retained correction history are not implemented in Core. |
| 8. Scheduled analysis and Insight generation | ⬜ Not started | Weekly, catch-up-safe, evidence-backed Insight generation is not implemented. |
| 9. Insight synchronization and Android inbox | ⬜ Not started | Durable Core-to-Android Insight delivery through Firestore is not implemented. |
| 10. FCM notification delivery | ⬜ Not started | Selective push signaling and device preferences are not implemented. |
| 11. Meal analysis assistance | 🟡 Partial / deferred | The replaceable Android `MealAnalyzer` boundary is merged; HTTP remains legacy/development only. No production on-device runtime/model is selected yet. |
| 12. Inventory module inside Android | ⬜ Not started | Uses the integration boundary established by meal contracts. |
| 13. Fitness and swimming domains | 🟡 Partial / deferred | Android training foundations exist in `main`; the cross-domain event, analysis and Insight completion gate is not implemented. |
| 14. Optional VPS services | ⬜ Not started | Added only for capabilities Android, Firebase and Core do not cover. |
| 15. Broader personal intelligence | ⬜ Not started | Photos, documents, projects and controlled automations. |

## Current focus

> Finish the reliable **offline nutrition record boundary on Android** before Firebase: verify and merge canonical meal IDs/revisions, normalize the remaining canonical component/source fields, and complete restart/correction verification. Stage 1 Windows verification remains outstanding in parallel; after the local Stage 3 gate is stable, continue with Stage 2 Firebase identity/security and the asynchronous event path.

This is a deliberate product-focus choice. Stage numbering still describes architectural capabilities and completion gates; it does not require every code change to be performed strictly in numeric order when an earlier local product dependency can be completed independently.

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
- initial Phase 1 implementation plan.

### Completion gate

A meal record and event batch can be represented in implementation-neutral JSON and validated against canonical schemas.

### Completion evidence

- `mosaic-docs` architecture PR #1 merged into `main`;
- `mosaic-contracts` Phase 1 contracts PR #1 merged into `main`.

---

## Stage 1 — Windows Core foundation — 🟡 Partial / verification required

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

The foundation from `mosaic-core` PR #1 is merged into `main`. This stage becomes ✅ **Complete** only after the clean Windows 11 completion check succeeds. Merge state and runtime verification are tracked separately.

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

### Boundary

Firebase begins after Android has a stable confirmed local record. Meal capture and daily progress must not depend on this stage. Raw meal photos and unconfirmed model estimates are not part of the default event relay.

---

## Stage 3 — Android meal recording and local dashboard — 🟡 Partial / verification required

### Goal

Provide a useful offline nutrition experience before Core or Firebase analysis is available.

### Repository

- `mosaic-android`

### Deliverables

- Room entities for meals and components;
- manual meal creation, editing and deletion/correction semantics;
- stable meal and component IDs;
- quantity, unit, preparation state and nutrition values;
- optional Inventory item references;
- daily protein and calorie totals;
- configured daily goals;
- deterministic remaining-goal calculation;
- local history by date;
- immediate updates after a meal is recorded or corrected;
- restart persistence;
- capture UI that does not require a model or Core connection.

### Current state — 2026-09-06

Merged into `mosaic-android/main`:

- Room-backed local meal persistence and daily nutrition totals;
- configurable local calorie/protein goals, consumed/goal/remaining progress and deterministic goal calculations from PR #18;
- fully offline manual meal capture from PR #19;
- the replaceable `MealAnalyzer` boundary, with the previous HTTP analyzer retained only as a legacy/development adapter, from PR #19;
- Fit unit tests in Android CI from PR #19;
- saved-meal editing and confirmed local deletion UI from PR #20;
- immediate recalculation of Today totals after save/edit/delete;
- physical-device smoke verification of the manual capture and saved-meal edit/delete flows.

Open Draft PR #21 — `Add canonical meal identity and revision persistence`:

- migrates Room v3 to v4 without requiring an uninstall or data reset;
- separates business `mealId` from analysis provenance `analysisId`;
- adds stable UUID `mealId` and `componentId` values;
- stores corrections as append-only revisions with `supersedesRevision` rather than replacing the previous row;
- stores deletion as a `deleted` tombstone revision rather than physically deleting history;
- preserves existing v3 meals by deterministic UUID migration;
- adds database revision-semantics tests to CI;
- CI passes database compile/tests, Fit tests, Photos compile and app assemble on the PR head;
- physical migration/restart verification against an existing device database is still required before merge.

### Remaining work for Stage 3

- verify the v3→v4 migration and revision/tombstone behavior on the physical Android device;
- review and merge PR #21 after successful device verification;
- complete canonical component normalization: numeric quantity, canonical unit, preparation state, per-component nutrition and nutrition status;
- persist canonical source/device metadata required by `MealRecord`;
- add exact canonical DTO serialization/validation before event-outbox work;
- complete restart verification after creation, repeated corrections and deletion;
- verify accurate totals and remaining goals after those restart cycles.

### Completion gate

A user can record and correct meals offline, restart the application, and immediately see accurate daily totals and the remaining amount toward the configured protein and calorie goals without Core, Firebase or a model call.

---

## Stage 4 — Android event outbox and Firebase publishing — ⬜ Not started

### Goal

Convert confirmed Android records into retry-safe immutable events and publish them to the authenticated user's Firestore event space.

### Repository

- `mosaic-android`

### Deliverables

- mapping between canonical Room records and contract DTOs;
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

Turn synchronized confirmed history into useful proactive intelligence without requiring the user to ask Core a question.

### Repository

- `mosaic-core`

### Deliverables

- scheduled analysis runner;
- catch-up behavior when the computer was offline;
- per-user analysis checkpoints;
- initial weekly nutrition summary;
- detected-pattern and recommendation-candidate outputs;
- evidence references to exact meals and revisions;
- clear distinction between stored facts, deterministic calculations and inference;
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

## Stage 11 — Meal analysis assistance — 🟡 Partial / deferred

### Goal

Reduce manual entry while preserving user control and the distinction between estimates and confirmed facts.

### Architecture direction

```text
Camera → MealPhotoInput → MealAnalyzer
                         ├─ on-device adapter (preferred production path)
                         └─ remote/HTTP adapter (development or explicit fallback)
                                  ↓
                           analysis estimate
                                  ↓
                         user review/correction
                                  ↓
                     canonical MealRecord
```

The capture UI must not depend on a specific model runtime. The runtime/model can be replaced without changing the canonical meal record or local dashboard.

### Deliverables

- photo- or text-assisted meal analysis;
- replaceable local or remote model adapter;
- preferred on-device production path for immediate mobile analysis on capable hardware;
- `meal-analysis` kept separate from canonical `MealRecord`;
- review, confirmation and correction flow;
- retained assumptions, confidence and original estimate;
- only confirmed output becomes a trusted revision;
- representative-device benchmarking before selecting a production runtime/model.

### Current state

- a tested server-side meal-analysis path exists from earlier work;
- the Android `MealAnalyzer` application boundary is merged in PR #19, with HTTP retained only as a legacy/development adapter;
- offline manual capture remains available regardless of analyzer availability;
- no on-device ML runtime or production meal model has been selected or bundled;
- model selection is intentionally deferred until representative Android hardware can be benchmarked for accuracy, memory, latency, battery and thermal behavior.

The adapter boundary is merged stage evidence, but the production on-device analyzer and the final estimate-to-canonical-record mapping remain incomplete.

### Completion gate

A generated estimate can be corrected and converted into a canonical meal record without losing the original estimate, assumptions or confidence, and normal capture does not require the home Core to be online.

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

## Stage 13 — Fitness and swimming domains — 🟡 Partial / deferred

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

- remote model inference when an on-device or Core-local option is inappropriate;
- external webhooks and integrations;
- scheduled cloud workflows that must run while Core is offline;
- media processing too expensive for the phone;
- controlled remote commands;
- vendor-independent relay if Firebase is later replaced.

The VPS must not silently become the canonical personal-intelligence database or the mandatory meal-analysis path.

### Completion gate

A concrete capability requires the VPS, has explicit authentication and retention rules, and can fail without corrupting Core's durable local state or blocking the Android offline experience.

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

The current product work intentionally completes the offline Android nutrition experience before Firebase. Stage numbers remain stable; this list reflects execution focus rather than renumbering the architecture.

```text
✅ 0. Architecture and initial contracts
🟡 1. Verify the merged Windows Core foundation on Windows 11
🟡 3. Verify/merge PR #21, then finish full canonical local MealRecord fields  ← current focus
🟡 Q. Fit tests are merged into CI; PR #21 adds database revision-semantics tests; Core/Contracts quality gates remain
⬜ 2. Establish Firebase Auth, Firestore paths and Security Rules
⬜ 4. Build the Android event outbox and Firebase publishing
⬜ 5. Consume and validate Firebase events in Core
⬜ 6. Persist events idempotently in Core
⬜ 7. Build meal projection and correction history
⬜ 8. Generate a weekly evidence-backed Insight
⬜ 9. Synchronize the Insight to the Android inbox
⬜ 10. Signal the Insight selectively through FCM
```

Stage 11 on-device meal assistance can advance incrementally after the Stage 3 local record boundary is stable. Its adapter boundary is already merged, but a production runtime/model is not required for Stage 3 completion.

## First useful product milestone

The first product milestone is reached when:

1. Android records and corrects meals offline and immediately shows daily calorie/protein totals and remaining configured goals.
2. Events synchronize asynchronously through Firebase.
3. Core catches up when the Windows computer becomes available.
4. Core generates one evidence-backed weekly Insight from confirmed data.
5. The Insight appears durably in the Android inbox.
6. An optional FCM notification points to that Insight without being required for delivery correctness.

On-device meal analysis improves capture convenience but is not required for this milestone's correctness.

## Working rule

The project has one primary implementation focus at a time. A stage is marked ✅ **Complete** only when its completion gate is verified and the relevant implementation is merged into `main`.
