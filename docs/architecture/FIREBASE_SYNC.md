# Firebase Synchronization Decision

## Status

Accepted for the planned Android-to-Core synchronization architecture.

Implementation has not started yet. This document defines the intended boundary before code is added.

## Decision

Use Firebase Authentication and Cloud Firestore as the initial cloud identity and event-relay layer between Mosaic Android and Mosaic Core.

```text
Android Room and outbox
        |
        v
Firebase Auth + Firestore
        |
        v
Mosaic Core local event store
```

Firebase is not the canonical Mosaic intelligence database. Core remains responsible for durable accepted history, projections, provenance, indexing and cited answers.

## Motivation

Direct Android-to-home-computer synchronization creates avoidable operational requirements:

- the home computer must be reachable;
- local network addresses may change;
- away-from-home synchronization requires NAT, VPN or a public API;
- the phone and Core may not be online at the same time;
- authentication and future multi-user separation become custom infrastructure.

Firebase provides an always-available rendezvous point while allowing Core to remain local-first.

## Responsibilities

### Firebase Authentication

- identifies Android users;
- provides a stable `uid`;
- enables future multiple users without inventing a password system;
- scopes Firestore Security Rules.

### Cloud Firestore

- receives immutable contract events from Android;
- buffers events while Core is offline;
- supports retry and offline-first mobile behavior;
- stores limited device and consumer synchronization state;
- separates user event spaces by `uid`.

### Mosaic Core

- consumes events only for explicitly configured users;
- validates all payloads against pinned `mosaic-contracts` schemas;
- rejects malformed and unsupported events;
- deduplicates by stable `eventId`;
- stores accepted events locally;
- builds projections and correction history;
- keeps provenance and citations;
- remains usable for already-synchronized data when Firebase is unavailable.

### Mosaic Android

- stores domain records in Room;
- creates stable IDs and revisions;
- creates immutable event documents;
- maintains a local retry-safe outbox;
- displays pending, synchronized and failed states.

## Event model

Events are append-only. A correction creates another event rather than replacing the previous business event.

Suggested Firestore path:

```text
users/{uid}/events/{eventId}
```

Suggested event metadata:

```json
{
  "eventId": "uuid",
  "eventType": "nutrition.meal.recorded",
  "eventVersion": 1,
  "aggregateId": "meal-id",
  "aggregateRevision": 1,
  "producerDeviceId": "device-id",
  "createdAt": "timestamp",
  "payload": {}
}
```

The exact wire structure remains owned by `mosaic-contracts`.

## Idempotency

Firestore delivery and Core polling or listeners may expose an event more than once. Core must enforce uniqueness locally by `eventId`.

A checkpoint improves efficiency but is not the correctness boundary. Re-reading old events must not duplicate accepted records or projection effects.

## Multiple users

The first usable installation may configure one Firebase user, but every cloud event and local accepted event must carry owner identity.

Core must not use a global unscoped event collection. Future household support may associate several Firebase users with one Core installation while applying separate permissions and provenance.

## Security

- Android access is restricted with Firebase Authentication and Firestore Security Rules.
- Rules must ensure a client cannot read or write another user's event path.
- Core credentials must be stored outside Git.
- Firebase server SDKs use IAM and bypass client Security Rules; Core must independently constrain which users it processes.
- Contract validation remains mandatory after download.
- Sensitive payload retention and optional application-level encryption require an explicit decision before syncing broader domains.

## Retention

Firestore should retain only the data required for reliable synchronization and the selected recovery policy.

A later policy must define:

- whether successfully consumed events remain indefinitely;
- whether they expire after every authorized Core acknowledges them;
- how a new Core installation performs recovery;
- how user deletion propagates;
- whether encrypted exports are maintained.

Until that policy is implemented, deletion must not occur automatically.

## Consequences

### Benefits

- the phone and Core do not need simultaneous availability;
- no direct public exposure of Core is required;
- away-from-home synchronization works naturally;
- mobile offline behavior is simpler;
- user identity and isolation have a clear foundation;
- the optional VPS can be deferred.

### Costs and risks

- synchronized records exist in a Google-managed cloud service;
- Firebase configuration, Rules and IAM become security-critical;
- the system gains vendor dependency;
- Firestore costs and quotas require monitoring as data volume grows;
- deletion, export and encryption policies must be designed carefully.

## Rejected alternatives for the first slice

### Direct local HTTP synchronization

Useful for debugging but not selected as the primary transport because it requires network reachability and complicates remote and multi-user operation.

### Mandatory Mosaic VPS relay

Remains possible later, but creates additional deployment, authentication, persistence and maintenance work that Firebase can initially cover.

### Firestore as the only database

Rejected. It would weaken the local-first architecture and couple projections, retrieval and long-term intelligence to a cloud database.

## Implementation order

1. Verify and merge the Windows Core contract foundation.
2. Create Firebase development configuration and Auth strategy.
3. Define and test Firestore Security Rules.
4. Add Android Firebase identity and immutable event publishing.
5. Add a Core Firebase consumer and local validation.
6. Add durable local event storage and deduplication.
7. Build meal projections, history, retrieval and citations.
