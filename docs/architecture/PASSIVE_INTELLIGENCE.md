# Passive Intelligence and Notifications

## Status

Accepted as the architectural interaction model for **deferred Mosaic Core**. The **current user-facing priority** is the on-device Android voice agent and its tools. See [voice-first decision](VOICE_FIRST_PRODUCT_DIRECTION.md).

Mosaic Core is not assumed to be continuously reachable from the phone. The home computer may be offline, asleep or disconnected, so the system must remain useful without treating Core as an always-available conversational server.

## Decision

Mosaic is split into two complementary experiences. "Passive Core" does not mean the phone itself must be passive: Android may provide an always-available local voice assistant while Core remains asynchronous.

```text
Android primary experience (active)
- explicit foreground voice turn (wake word later)
- local Hebrew ASR → Gemma tool selection → typed result → BlueTTS
- permission-scoped Android tool access, beginning with recent WhatsApp notifications
- truthful no-access / no-notification handling

Future optional domains
- nutrition, photos, workouts and other local data accessed by tools
- deterministic calculations and user-confirmed records

Mosaic Core asynchronous intelligence
- periodic synchronization
- historical and cross-domain analysis
- proactive insights and recommendations
```

Android owns the immediate operational experience. Core periodically analyzes synchronized confirmed history and publishes durable insights back through Firebase.

This separation is intentional: model-assisted meal capture may use a local model on the phone, while deeper weekly pattern analysis remains a Core responsibility.

## Immediate Android capabilities — current focus

Mosaic should run useful actions from natural Hebrew requests through **real, explicit Android tools**, not require a separate nutrition or photos app workflow first. The first tool reads the latest *accessible* WhatsApp notification with user-granted Notification Listener access; it does not scrape WhatsApp's database or guarantee full message history. It must return structured results and distinguish permission denied, unavailable and empty cases. The answer is spoken locally.

Previously implemented Room-backed nutrition, workout and photo capabilities are preserved for possible future tool adapters, but are not active milestones. Deterministic totals continue to belong in code/SQL if those tools are built.

## On-device voice assistant

Mosaic may provide an always-available local assistant experience on Android without keeping a general-purpose LLM active continuously.

Preferred **eventual** runtime shape (initial voice turn is foreground/user-triggered; wake word comes later):

```text
user activation or future low-power wake-word detector ("Mosaic")
  ↓
local ASR
  ↓
local agent / tool router
  ↓
permission-scoped local tools
  ↓
local TTS
```

Rules:

- wake-word detection is a separate lightweight component from ASR and the agent model;
- ASR, the agent model, vision and TTS should run only when required by an active interaction;
- the assistant must remain functional for supported local tasks while Core and Firebase are unavailable;
- notification/message access requires explicit Android permission and remains local by default;
- reading content aloud is a retrieval operation, not authorization for a reply or other side effect;
- externally visible or destructive tool calls are gated by an action policy and confirmation where appropriate;
- Android-supported assistant/background APIs are preferred over attempts to keep an unrestricted microphone service alive.

A representative interaction is:

```text
"Mosaic"
→ "Yes?"
→ "Read my latest message"
→ query authorized recent notifications
→ select the requested item
→ speak it through TTS
```

The local assistant may later use the same tool boundary for photo search, local memory retrieval, calls, reminders and application navigation.

## On-device meal-analysis assistance (deferred optional tool)

Meal-analysis assistance is an optional enhancement to capture, not a replacement for trusted records.

Preferred production flow on a capable Android device:

```text
Camera
  ↓
MealPhotoInput
  ↓
MealAnalyzer
  ↓
on-device model runtime
  ↓
estimated foods / quantities / nutrition
  ↓
user review or correction
  ↓
confirmed local meal record
  ↓
Room + deterministic daily totals
```

Architectural rules:

- the capture UI depends on a replaceable `MealAnalyzer` boundary rather than a specific model runtime;
- an on-device analyzer is the preferred production path for immediate photo analysis;
- a remote or HTTP analyzer may exist for development or an explicit fallback, but it must not be required for normal meal capture;
- the home Core being offline must not block manual or supported on-device capture;
- model output is an estimate until confirmed by the user;
- assumptions, confidence and original estimated values should be retained when available;
- only confirmed structured meal data becomes trusted history and enters the normal event pipeline.

Model/runtime selection is intentionally deferred until representative target hardware can be benchmarked for accuracy, memory use, latency, thermal impact and battery cost.

## Core responsibilities

Core is primarily an asynchronous personal-intelligence engine. It should:

- consume new immutable events when the computer is available;
- update durable local projections and history;
- run scheduled or catch-up analysis jobs;
- detect trends, recurring patterns and anomalies;
- produce summaries, insights and recommendation candidates;
- preserve evidence and provenance for every generated result;
- publish selected results to Firebase for Android consumption.

Core is not required to analyze a meal photo in the moment. Its value is deeper historical and cross-domain analysis over confirmed data.

Interactive querying of Core may be added when Core is reachable, but it is not required for the Android local assistant. Ordinary wake-word, ASR, local retrieval/tool use and TTS should not depend on Core availability.

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
Android confirmed events
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

## Voice-first first milestone and deferred Core boundaries

The current first useful slice is: a reliable local Hebrew spoken turn; a permission-checked, read-only notification tool selected through the agent; and a local spoken reply (or explicit no-access/no-message result). No cloud or Windows Core dependency.

Core's original first passive slice — syncing meals, calculating weekly nutrition Insights, delivering them through Firestore and signaling with FCM — is **deferred**, not complete or deleted. When revived, the durable-before-notification, idempotency, provenance and per-user isolation rules in this document still apply. Wake-word activation and additional side-effecting actions are separate milestones.
