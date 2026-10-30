# Security and Permissions

## Trust boundaries

Mosaic treats the home node, Android clients, on-device model runtimes, VPS workers, cloud model providers and external connectors as explicit trust zones.

An on-device model runtime executes inside the Android application boundary. Remote model providers and the home Core remain separate destinations and must never be assumed to be available for immediate mobile workflows.

## Data classification

Each source and artifact is classified as private, sensitive or restricted. Policy checks consider user identity, device identity, requested operation, destination model, connector scope and data classification.

Meal photos and nutrition records are sensitive personal data. Notification contents, message previews, transcripts and voice interaction history are also sensitive local context. Immediate meal capture and supported local voice interactions must remain functional without transmitting those inputs to the home computer or a cloud model.

## Core controls

- least-privilege connector scopes;
- per-device authentication and revocation;
- encryption in transit and at rest;
- explicit approval for destructive or externally visible actions;
- immutable audit records for tool calls, remote jobs and memory changes;
- scoped remote-job payloads with expiry;
- source-aware authorization before retrieval;
- secrets outside source control.

## Model routing

Sensitive data defaults to local processing. A locally available tool is not automatically safe to invoke: authorization is evaluated per operation, destination and side effect.

For meal capture specifically:

- manual entry is fully local;
- photo-assisted analysis should use an on-device model adapter on capable Android devices;
- an analysis result is an estimate, not a trusted fact, until the user reviews or confirms it;
- a remote HTTP/model adapter may exist for development or an explicitly selected fallback, but it is not required for the production capture path;
- remote processing is permitted only through an explicit policy that minimizes payloads and records the provider, purpose, model and retention assumptions.

## Local tool execution boundary

The on-device voice agent is the primary user-facing product, but **the LLM is not the authority**. Every tool has a defined ID/version, typed input and result, explicit Android capability checks, bounded execution, and a read-only or side-effecting classification. The dispatcher validates requests before calling deterministic Android code. Unavailable permissions/content produce errors; no fabricated result or silent cloud fallback is allowed.

Initial tool: `notifications.read_latest_whatsapp` is **read-only** and can access only WhatsApp notification content currently available through Android's granted Notification Listener surface. It cannot guarantee message history, hidden text, or app-private conversations. Notification texts and transcripts remain local by default; diagnostics should avoid raw content. No replies, sends, deletes or other effects are implied. Screen lock, Bluetooth and notification privacy rules still apply.

Nutrition and photo features are paused as first-class product work. If later made available as tools, they require their own explicit permissions, data minimization, user confirmation for mutations and retention policy; no broad automatic gallery scan or food data upload is introduced by this pivot.

## Android voice and notification handling

The local assistant uses least privilege:

- microphone access requires explicit Android permission and is used only for the declared assistant/wake-word behavior;
- notification access requires explicit user enablement through Android's notification-listener permission surface;
- notification-derived message text remains local by default and is not automatically synchronized to Firebase or Core;
- wake-word detection should minimize retained audio and should not persist arbitrary background microphone audio;
- ASR transcripts should be ephemeral unless a product feature explicitly requires retention;
- reading a message aloud does not authorize replying to it;
- sending messages, deleting data, purchases, account changes and similar external/destructive actions require a separate tool permission/policy decision and confirmation when appropriate;
- lock-screen and Bluetooth playback must respect privacy settings and avoid exposing sensitive content without the user's requested interaction.

If Mosaic is selected as the device assistant, supported Android voice-interaction APIs should be used rather than bypassing background execution restrictions.

## Voice v0.1 screen-off consent and responsiveness (2026-10-30)

Hands-free standby is **opt-in**, not covert background recording. User selection of Mosaic as the Android assistant and microphone/notification grants happen through OS-controlled surfaces; a persistent Stop action must end microphone capture and service lifetime. Never attempt unauthorized background microphone start, boot-triggered microphone FGS startup or keyguard bypass. Wake waiting audio is transient/bounded and is not retained or uploaded. Diagnostic timing/status may be recorded without raw idle audio, private transcripts or notification payloads.

Fixed acknowledgement/ready/processing cues are app-controlled local audio, not model output or authorization. Do not claim “ready” until the actual capture path is open; do not speak private notification content through a locked device. A recognized wake phrase is not speaker authentication. Battery/thermal and false-wake impacts of the CPU Zipformer listener must be measured and disclosed before calling the release suitable for daily use. See [v0.1 release gate](../releases/VOICE_V1_HANDSFREE_ACCEPTANCE.md).

## Android media handling

Meal photos captured for analysis should remain in application-controlled temporary storage unless the user explicitly chooses otherwise.

- temporary photos are not written to the public gallery by default;
- temporary files should be removed on an appropriate retention schedule;
- image bytes are not included in synchronization events by default;
- only confirmed structured meal data is expected to enter the normal event pipeline;
- adding a remote analyzer later requires an explicit privacy and retention review.

## Memory safety

Model-generated memories begin as candidates unless produced by a trusted deterministic rule. User corrections take precedence, contradictions create new versions rather than silent overwrites, and every active memory remains traceable to evidence.

The same trust rule applies to meal analysis: model-estimated foods, quantities and nutrition remain suggestions until confirmed by the user.

## Remote workers

VPS jobs receive only the required input, use short-lived credentials, store temporary encrypted data, and delete it according to an auditable retention policy. The VPS never receives broad filesystem access to the home node.

## Secrets

Use OS keychains or a dedicated secrets manager for the initial Windows and Android deployments. Maintain separate credentials per connector and environment, and support independent rotation and revocation. Container-specific secret mechanisms may be used only for optional services that actually run in containers later.
## Headset route consent and privacy

Headset wake requires explicit visible opt-in and microphone permission; pairing
or wearing buds is not consent or user authentication. Retain the persistent
Stop control, no idle PCM/transcript retention and existing locked-message refusal.
Do not persist Bluetooth names, addresses or raw audio in diagnostics.

Before enabling the route adapter declare MODIFY_AUDIO_SETTINGS; microphone FGS
type/permissions and RECORD_AUDIO apply to the recording owner. Request
BLUETOOTH_CONNECT through Android only if paired-device Bluetooth APIs need it;
do not add scan/location access for AudioManager route selection. Bluetooth
connectivity does not waive background microphone start, while-in-use or boot
restrictions. Android 15 focus eligibility requires the top app or a running FGS.
See [FGS types](https://developer.android.com/develop/background-work/services/fgs/service-types)
and [Bluetooth permissions](https://developer.android.com/develop/connectivity/bluetooth/bt-permissions).

Lost route, call/focus interruption, revocation or Stop must cease capture and
private playback before cleanup. Never silently fall back to phone microphone or
speaker; reconnect requires fresh route verification. The initial unwired lease
changes no grants, enabled behavior, retention or signed IPC contract. Physical
Pixel + Pixel Buds testing remains required before claiming screen-off support.
