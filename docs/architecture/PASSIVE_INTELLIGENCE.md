# Passive Intelligence and Notifications

## Status

Accepted as the primary interaction model for Mosaic Core.

Mosaic Core is not assumed to be continuously reachable from the phone. The home computer may be offline, asleep or disconnected, so the system must remain useful without treating Core as an always-available conversational server.

## Decision

Mosaic is split into two complementary experiences:

```text
Android local experience
- immediate calculations
- offline dashboard
- current goals and progress

Mosaic Core asynchronous intelligence
- periodic synchronization
- historical and cross-domain analysis
- proactive insights and recommendations
```

Android answers simple deterministic questions from local Room data. Core periodically analyzes synchronized history and publishes durable insights back through Firebase.

## Immediate Android capabilities

The Android application should calculate locally, without requiring Core or Firebase:

- protein consumed today;
- protein remaining against the selected target;
- recorded calories and other nutrition totals;
- latest weight and measurements;
- recent meals and workouts;
- basic goal progress.

These calculations are deterministic projections over local records. They do not require a language model.

## Core responsibilities

Core is primarily an asynchronous personal-intelligence engine. It should:

- consume new immutable events when the computer is available;
- update durable local projections and history;
- run scheduled or catch-up analysis jobs;
- detect trends, recurring patterns and anomalies;
- produce summaries, insights and recommendation candidates;
- preserve evidence and provenance for every generated result;
- publish selected results to Firebase for Android consumption.

Interactive local querying may be added when Core is reachable, but it is not required for the main user value.

## Insight model

An insight is a durable result, not a notification payload.

Suggested Firestore path:

```text
users/{uid}/insights/{insightId}
```

An insight should include:

- stable `insightId`;
- owner `uid`;
- type and schema version;
- title and concise summary;
- generated time and analysis period;
- importance or notification level;
- evidence references;
- generating Core installation and analysis-run identifiers;
- read, dismissed or archived state where appropriate;
- expiration or supersession metadata when relevant.

Examples include:

- weekly nutrition summary;
- recurring protein shortfall on swimming days;
- training-load pattern;
- Inventory expiry warning;
- newly indexed photo or document result;
- recommendation candidate requiring user review.

## Analysis cadence

The initial Core workflow should support both:

### Startup catch-up

```text
Core starts
→ downloads unprocessed events
→ validates and stores them
→ updates projections
→ runs due analysis jobs
→ publishes new insights
```

### Periodic analysis

The default initial cadence is weekly. Daily summaries may be added later where they create clear value.

The cadence must be configurable per user and per insight category. A missed schedule must be catch-up safe: Core may run the pending analysis when it next becomes available.

## Firebase result path

```text
Android events
    ↓
Firestore event relay
    ↓
Mosaic Core analysis
    ↓
Firestore insights
    ↓
Android local insight inbox
```

Firestore is the synchronization and delivery layer for generated insights. Core remains the durable analysis and provenance authority.

## Firebase Cloud Messaging

Firebase Cloud Messaging is used only as a lightweight signal that a durable insight is available.

```text
Core writes insight to Firestore
        ↓
trusted notification dispatcher sends FCM
        ↓
Android receives notification
        ↓
user opens the stored insight
```

The notification should contain only minimal routing data, such as:

- `insightId`;
- destination screen;
- optional category;
- short privacy-safe title and body.

The full analysis, evidence and recommendation must remain in the Insight document and local Android storage, not in the FCM payload.

## Notification dispatcher

Notification sending must occur from a trusted environment.

The preferred long-term design is:

```text
Core creates Firestore Insight
→ Cloud Function or equivalent trusted dispatcher
→ FCM delivery to registered user devices
```

A direct Core-to-FCM sender may be used initially if it simplifies the first implementation, but it must use an outbox or retry-safe delivery record so an Insight is not lost when notification delivery fails.

## Device registration

Each Android installation registers separately:

```text
users/{uid}/devices/{deviceId}
```

Device metadata should include:

- installation identifier;
- platform and application version;
- current FCM registration token;
- token update time;
- notification permission state;
- last-seen time;
- enabled insight categories.

Invalid or expired registration tokens must be removed or disabled without affecting the underlying Insight.

## Notification policy

Not every Insight should generate a push notification.

Suggested levels:

- `silent` — stored in the application only;
- `notable` — normal notification;
- `important` — prominent notification for a genuinely significant result.

The policy must support:

- opt-in or opt-out by category;
- quiet hours;
- maximum notification frequency;
- weekly digest instead of individual notifications;
- deduplication by `insightId` and notification attempt;
- suppression of obsolete or superseded insights;
- privacy-safe lock-screen text.

Health-related messages must avoid diagnosis or unjustified certainty. The detailed Insight should expose evidence, assumptions and confidence.

## Reliability

FCM delivery is not the correctness boundary.

- An Insight is considered published only after it is durably written to Firestore.
- A notification may be delayed, duplicated or never displayed.
- Android must synchronize the Insight inbox independently of push delivery.
- Opening a notification should resolve the referenced Insight by ID.
- Notification delivery attempts should be idempotent and auditable.

## Multiple users

Insights and device registrations are always user-scoped. A household Core installation may analyze several explicitly authorized users, but it must publish each Insight only to the correct user's path and authorized devices.

Shared or household insights require an explicit audience model rather than copying one user's private result into another user's namespace.

## Phase 1 boundaries

The first useful slice should prove:

1. Android calculates current daily nutrition totals locally.
2. Core can synchronize when available without being publicly reachable.
3. Core generates a weekly evidence-backed nutrition Insight.
4. The Insight is written to the correct Firebase user path.
5. Android receives and stores the Insight.
6. An FCM notification can open the exact stored Insight.
7. Reprocessing or retrying does not create duplicate Insights or duplicate notification records.

Interactive remote question answering, real-time Core availability and autonomous high-frequency notifications are explicitly deferred.
