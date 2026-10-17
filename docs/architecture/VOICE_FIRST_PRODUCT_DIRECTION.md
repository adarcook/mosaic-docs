# Voice-first, tool-first product direction (2026-10-17)

**Status:** proposed in a Draft PR; not an accepted `main` decision until merged.

## Decision

Mosaic's **active product** is a Hebrew-first, on-device Android personal voice agent. The user speaks, local ASR transcribes, a local model such as Gemma interprets intent and chooses a **permission-scoped tool**, and local TTS speaks a result. The Android experience must remain useful with the home Windows computer and Firebase offline.

The previous product-led nutrition and photo/gallery modules are **paused as primary user-facing work**. They are not prerequisites to the agent, and the current milestone does not include meal capture, photo galleries, nutrition dashboards, swimming dashboards, Firebase integration or Windows Core analysis. Preserve existing merged code, contracts and records; do not delete or break them merely because priorities changed. In the future, selectively expose valuable domain operations through the same tool interface (for example `nutrition.get_daily_protein` or `photos.search_local`) if the user wants them.

This is a *priority and interface pivot*, not completion of the prior nutrition roadmap, nor a claim that Android grants unrestricted access to device data.

## Local agent path

```text
User activates foreground voice session (wake word later)
  → local Hebrew ASR (Zipformer candidate; Whisper fallback)
  → local Gemma tool planning / argument preparation
  → explicitly authorized Android tool dispatcher
  → typed tool result + controlled error handling
  → local BlueTTS response
```

Initial interaction uses foreground, user-triggered recording. Wake word is a later slice and must respect Android assistant/background APIs. Do not keep large ASR/LLM/TTS engines running continuously.

A local tool is an **execution boundary**, not just a prompt or a model hallucination:
- fixed tool ID, description, version, typed arguments and result;
- explicit capability/permission checks and clear failures;
- side-effect category (read-only vs externally visible/destructive);
- deterministic Android implementation, not LLM-simulated Android state;
- bounded execution, cancellation, audit/diagnostics without exposing private content;
- no automatic upload of message text, audio or transcripts to Firebase/Core.

Model output must not itself grant permissions or justify sensitive writes. Sender/contact resolution must be disambiguated before any future outward action.

## First meaningful capability

**Read the last accessible incoming WhatsApp notification aloud**, only after the user enables Android notification access and requests it.

Acceptance sequence:
1. Run repeatable local Hebrew spoken turns on the Pixel without crash, unbounded memory growth or missed final words.
2. On an explicit speech request, the agent selects a **read-only** `notifications.read_latest_whatsapp` tool and receives a structured result (or a clear unavailable/permission-denied result).
3. The tool reads only locally accessible notification content in its granted scope. Android Notification Listener cannot guarantee full chat history or messages absent from accessible notifications; do **not** claim arbitrary WhatsApp database access.
4. The assistant speaks a concise answer through local TTS without transmitting message content to a remote system.
5. Confirm behavior with no notification, revoked permission, screen lock/privacy preferences, cancellation, repeated turns and safe worker shutdown.

No sending, replying, deleting or external action is included. Those require separate tool implementations, policy gates and appropriate user confirmation.

## Status discipline and retained architecture

- As of 2026-10-17, Stage 0 is the last `Complete` stage in `main`. The Windows Core Stage 1 completion check remains open, though it is **not the immediate product focus**.
- Experimental voice PRs in `mosaic-android` remain draft/unmerged; one successful voice turn is not a general reliability gate.
- Existing nutrition/photo implementation is retained as optional reuse material. Do not recast paused work as `Complete` or delete existing canonical contracts.
- The future Android event outbox → Firebase → Windows Core → evidence-backed Insights architecture remains a possible *asynchronous enrichment path*, not a prerequisite for local agent tools. FCM remains a notification signal.
- Do not add a VPS, expose home Core to the Internet, or replace Android permission and deterministic logic with LLM guesses to support the first tool.

## Next small slice

Repository: `adarcook/mosaic-android`.

After verifying the current voice candidate in multiple real Pixel turns, implement the minimal read-only Android notification tool boundary and permission/on-device tests, then wire its structured result into the local spoken assistant in a subsequent reviewable slice. Keep work in Draft PRs; merge only on explicit request.
