# Mosaic Voice v0.1 — Hands-free release contract

**Decision date:** 2026-10-30. **Status:** agreed product direction; implementation and physical release acceptance pending.  
**Owner-visible promise:** Once the owner explicitly enables listening and configures Mosaic as the Android assistant, they can wake Mosaic without touching the Pixel, receive an immediate audible acknowledgement, give a request after a genuine ready cue, and receive an on-device answer while the screen stays off. This is a private **personal beta**, not a claim of production reliability or zero battery impact.

## What is in v0.1

- One local opt-in wake phrase: **"מוזאיק"**, including documented recognizer variants. Wake detection is not speaker authentication.
- Wake and turn flow while the Pixel 10 Pro is screen-off / keyguard showing; no repeated touch or unlocking required for permitted, non-private turns **after explicit setup/arming**. Android may require rearming after reboot, force-stop, OS permission changes or assistant-role changes. Never silently bypass these constraints.
- **Immediate, engine-independent audio feedback.** A fixed verified local PCM clip or tone signals a detected wake; Gemma and BlueTTS do not load just to acknowledge. A second cue must announce actual microphone/ASR readiness, not simply that warm-up has started. A fixed “received / processing” cue may play after the captured request while heavy models work. Do not speak “ready” before the capture path is truly usable.
- Typed, permission-scoped **read-only** notification tool; only the latest accessible WhatsApp notification, not WhatsApp history. Locked/private content is withheld. Spoken response is local.
- Explicit stop control in a persistent notification, clear listening state and errors, bounded timeouts, cancellation and automatic re-arm after successful turns.
- Heavy ASR/LLM/TTS engines remain on demand, sequential where memory requires it, and are released safely. **Do not solve perceived latency by permanently pinning Gemma in RAM.**
- No Internet, Firebase, Windows Core, VPS, new domain tools, sending messages, replies or destructive device actions are required.

## Interaction state and feedback semantics

```text
OFF (no microphone)
  -- user opt-in + Android assistant selection / permissions -->
ARMED (local wake listener + visible Stop)
  -- verified local wake event --> ACK (play short pre-rendered feedback promptly)
  --> HANDOFF / PREPARING (release listening recognizer, prepare turn recording)
  --> READY (recording is actually possible; only now play "speak" cue)
  --> CAPTURING (user's request)
  --> RECEIVED (optional pre-rendered feedback, not BlueTTS)
  --> ASR / GEMMA / TYPED TOOL / TTS
  --> REARM (or controlled ERROR / STOP)
```

Do not drop speech because a “ready” cue preceded microphone readiness. Avoid duplicate cues and echo/self-wakes. A negative/failed stage must lead to an honest short error cue or visible status, not silent waiting or a fabricated answer. All processing remains offline and private by default.

## Android and privacy boundary

- Android background microphone restrictions still apply. The existing integration uses an explicitly selected `VoiceInteractionService`, signed Zipformer companion and microphone/media-playback foreground service. Those are implementation candidates, **not** guarantees that arbitrary background startup or hardware DSP hotword detection is available.
- Standby currently performs **Zipformer ASR on CPU** to match the wake phrase, not a measured low-power dedicated keyword engine. Measure power/thermal effects before daily-use acceptance; never claim a negligible battery cost from API shape alone.
- Setup is visible and optional; permissions use Android dialogs. Include a persistent listening notification and a working Stop action. No launch of a microphone FGS from a boot receiver or unlawful background context. No hidden always-on recording.
- Wake-audio windows remain volatile and bounded; no idle microphone audio/transcripts are saved, uploaded or used for unrelated features. The explicit request stage may retain its existing local private journal; no raw content in telemetry, CI or GitHub.
- Do not bypass keyguard. With the device locked, `notifications.read_latest_whatsapp` must refuse private message readout unless a separately reviewed, explicit privacy policy permits it. A successful wake does not authorize content disclosure.
- Explicitly show the user when Mosaic replaces the phone's default assistant and how to stop/revoke listening.

## Physical release acceptance — Pixel 10 Pro (not yet complete)

1. **Setup and stop:** Confirm Android assistant role, runtime permissions, signed companion and opt-in; persistent notification Stop actually stops the microphone. Confirm no automatic restart after forced stop.
2. **Screen-off wake:** From a verified screen-off, locked, untouched device, trigger several successive wake/answer/re-arm cycles. Do not infer screen-off wake solely from a later status snapshot.
3. **Acknowledge quickly:** Log monotonic timestamps for confirmed wake event, first fixed-cue playback request and playback initiation. **Target:** P95 under 500 ms after confirmed detection across at least 20 trials; label this a target until measured. Separate detector recognition delay, audible speaker delay and engine loading; never label a callback timestamp as measured human-audible onset.
4. **Real readiness:** The “speak now” cue must follow verified capture readiness. Try speaking immediately after the cue; no truncated first syllable. On failure, play an error cue and return to a known state.
5. **Actual work:** ASR → model-selected read-only tool → typed native result → local TTS on a real allowed notification; test no notification, denied/revoked permission, privacy lock, cancellation and negative requests.
6. **Reliability:** At least 10 consecutive full screen-off attempts without crashes, lost foreground-service promotion, stuck spinner, false “ready” or failure to re-arm. Test interruption by phone call/audio focus, process death, and repeated wake while busy.
7. **Power/quality:** Record representative idle battery baseline vs enabled wake listener across a sustained screen-off interval, CPU/thermal indications, missed/false wakes and impact on media/audio routing. **No battery or accuracy claim until measured**; decide the acceptable drain from measured user experience.
8. **Build/release:** CI passes on the exact commit; publish build identity and signer-compatible, data-preserving Pixel installation record. No app-data/model deletion, no credential exposure, no claimed physical pass without real device results.

## Development and merge order

- **Already merged:** Android Voice 0.22 baseline in `main` via PR #49, including experimental wake flow, signed wake companion and fixed PCM feedback. This is a *prototype baseline*, not v0.1 acceptance.
- **Existing stacked open work:** #50 (typed routing) → #51 (personal phrase dictionary) → #52 (model-selected typed tool). These should be reviewed, integrated in dependency order and merged only after validation of the combined head; passing synthetic 5/5 does not substitute for live end-to-end tests.
- **Next small implementation slice:** Add exact latency markers and correct wake-ack / actually-ready sequencing. Build and observe on Pixel before further wake/service refactoring.
- **Then:** Stabilize full locked screen-off turn/re-arm and measure power; fix only observed regressions. The optional Pixel Bridge #47 is not a runtime/release prerequisite. Do not indiscriminately merge old experimental draft PRs.
- Tag/publish **v0.1 personal beta** only after all release gates are substantiated. Never mark the historical numbered roadmap stages Complete merely because this independent voice milestone ships.

## References

- [Mosaic voice-first decision](../architecture/VOICE_FIRST_PRODUCT_DIRECTION.md)
- [Roadmap](../../ROADMAP.md)
- Android working notes: `mosaic-android/docs/handsfree-assistant.md`, `docs/screen-off-voice-lifecycle.md`, `docs/model-tool-agent.md`.
- [Android VoiceInteractionService](https://developer.android.com/reference/android/service/voice/VoiceInteractionService)
- [Android background microphone restrictions](https://developer.android.com/develop/background-work/services/fgs/restrictions-bg-start)
