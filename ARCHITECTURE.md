# Mosaic Architecture

## System overview

```text
Mosaic Android
  ├── Room / local domain state
  ├── local outbox and offline work
  └── Firebase Auth + Cloud Firestore
                    |
                    | versioned immutable events
                    v
              Firebase relay
                    |
                    | pull / listen when available
                    v
Mosaic Core on Windows 11
  ├── contract validation
  ├── durable local event store
  ├── domain projections and history
  ├── retrieval and provenance
  └── cross-domain intelligence
```

Firebase is the online identity and synchronization layer. It is not the durable personal-intelligence database and does not replace Mosaic Core.

## Component roles

### Mosaic Android

Owns mobile workflows and local operational state. It remains useful offline, stores domain records in Room, creates versioned contract events, and publishes immutable events to the authenticated user's Firestore space when connectivity is available.

Android does not need a direct network route to the home computer. Firestore allows it to record data while Core is offline and synchronize later.

### Firebase Authentication

Provides stable user identity and sign-in for Android. The Firebase `uid` is the external identity used to partition cloud synchronization data.

Firebase identity must be mapped explicitly to Mosaic users or household members. A Firebase account does not automatically grant access to every local Core profile.

### Cloud Firestore

Acts as a cloud event relay and synchronization buffer.

Firestore stores versioned, immutable events and limited synchronization metadata. It is not the canonical projection database, model memory, vector store, or long-term source of every Mosaic artifact.

The preferred shape is conceptually:

```text
users/{uid}/events/{eventId}
users/{uid}/devices/{deviceId}
users/{uid}/consumers/{coreId}
```

Corrections create new event revisions instead of overwriting prior events. Core remains idempotent even if an event is delivered more than once.

### Mosaic Core

Runs directly on the Windows 11 home computer and provides durable personal intelligence.

Core:

- consumes contracts through a pinned `mosaic-contracts` Git submodule;
- reads events belonging to configured Firebase users;
- validates every batch, envelope and typed payload locally;
- rejects unsupported or malformed contract versions;
- stores accepted events idempotently in a local database;
- builds current projections while retaining revision history;
- preserves provenance and exact source references;
- indexes trusted data and produces cited answers;
- remains the durable intelligence layer even if Firebase is replaced later.

### Mosaic Contracts

Defines implementation-neutral, versioned payloads. JSON Schema is initially canonical. Kotlin and Python bindings may later be generated from these definitions.

Firebase transports contract payloads but does not define them. Firestore Security Rules may enforce coarse ownership and shape constraints, while full contract validation remains in application tests and Mosaic Core.

### Mosaic Server

The VPS is optional and deferred. It may later provide model inference, webhooks, integrations, remote commands, or workflows that Firebase alone does not handle well.

It is no longer required for the first Android-to-Core synchronization path.

## Data lifecycle: confirmed meal

1. The user creates or confirms a meal in Mosaic Android.
2. Android stores the meal and its components locally in Room.
3. Android creates a versioned immutable event using `mosaic-contracts`.
4. The event enters a retry-safe local outbox.
5. Firebase Authentication identifies the user.
6. Android writes the event to `users/{uid}/events/{eventId}` in Firestore.
7. Firestore retains the event while Mosaic Core may be offline.
8. Core reads the event for an explicitly configured user.
9. Core validates the envelope and typed payload against its pinned contracts.
10. Core stores the original accepted event idempotently and preserves source metadata.
11. Core updates the meal projection and revision history.
12. Retrieval and calculations can cite the exact event and meal revision used.

## Correction lifecycle

A correction does not destructively replace the earlier synchronized event.

```text
nutrition.meal.recorded  revision 1
nutrition.meal.updated   revision 2
```

Core selects the latest valid revision for the current projection while retaining the complete history. This avoids relying on Firestore's document overwrite semantics for business history.

## Source-of-truth boundaries

### Android Room

- mobile workflow state;
- offline records;
- pending synchronization outbox;
- user-visible synchronization status.

### Firebase

- authenticated user identity;
- cloud relay of immutable events;
- device and consumer synchronization metadata;
- temporary or policy-controlled availability while Core is offline.

### Mosaic Core

- durable accepted event history;
- normalized projections;
- provenance and citations;
- indexes, memory and cross-domain intelligence;
- local retention and backup policy.

## Multi-user direction

Every event is scoped to a Firebase `uid`, and Core stores the owner identity with the event and projection.

The first implementation may configure one user. The data model and paths must nevertheless avoid a hard-coded global user so additional users or household members can be added without redesigning synchronization.

A future household model may map multiple Firebase users to one local Core installation while preserving per-user permissions and provenance.

## Trust boundaries

- Personal records synchronized through Firebase are cloud data and must be treated accordingly.
- Security Rules must restrict Android clients to their authorized user paths.
- Server SDKs and service accounts use IAM and must not rely on client Security Rules as their only authorization layer.
- Core must verify the configured user identity and validate every contract payload locally.
- Secrets, Firebase service credentials and local encryption keys never belong in Git.
- Model output remains an estimate until confirmed by the user.
- Mosaic Core must retain provenance for imported, calculated and inferred data.
- Sensitive payload encryption and cloud retention rules must be decided before broader personal domains are synchronized.

## Deployment direction

- Mosaic Android: Android device, Room and Firebase client SDKs.
- Firebase Authentication: cloud identity provider.
- Cloud Firestore: online immutable-event relay and limited sync metadata.
- Mosaic Core: Windows 11 home computer, direct Python installation and local storage.
- Mosaic Server: optional later VPS services for capabilities not covered by Firebase or Core.

## Open decisions

- Initial Firebase sign-in provider.
- Firebase project environments for development and production.
- Core authentication method: restricted service account, user-scoped tokens, or a small trusted relay.
- Firestore Security Rules and emulator test strategy.
- Cloud event retention, deletion and export policy.
- Whether sensitive event payloads require application-level encryption.
- Household and multi-user permission model.
- Conflict rules for concurrent revisions of the same aggregate.
- Contract generation strategy for Kotlin and Python.
- Initial meal-analysis adapter implementation.