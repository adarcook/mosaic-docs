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