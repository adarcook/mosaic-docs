# Mosaic development workflow — GitHub CI first

**Decision date:** 2026-10-17. **Applies to:** any future implementation slice, installable Android APK/version, Core or tool change. It is a **required delivery gate**, not a claim of real-device acceptance.

## Repository authority

GitHub is the source of truth. At the beginning of each development task, inspect the latest `main`, the relevant open/recently merged PRs and [ROADMAP](../ROADMAP.md). Do not duplicate an open slice or mark any stage Complete until its verified work is merged to `main`. Preserve existing branches, installed app data, local model files, credentials and signing boundaries.

## Mandatory GitHub-first build and test gate

1. Make the smallest testable change on a dedicated branch; push code and an accurately described **Draft PR**. Never merge without an explicit user request.
2. **Before asking the user to build, compile, install or debug a new version locally**, trigger or use a relevant **GitHub Actions workflow** for **that exact PR head commit SHA**. Where no suitable workflow exists, add one, or explicitly report why GitHub-hosted compilation is infeasible and what was verified instead.
3. For Android voice/Tool changes, run relevant Gradle `testDebugUnitTest` and `assembleDebug` or `compileDebugJavaWithJavac` for the **changed module(s)**, with pinned dependencies and model-free CI inputs. Where the relevant project depends on a companion APK/IPC contract, compile/test its integration boundary as well. Follow up with static/package integrity checks when available.
4. Inspect the actual **run conclusion and job steps** on the *latest head*, not only a previous green commit or a PR badge. If a run is pending/failed, **do not send the user to repeat the build**. Investigate logs, correct the code, push another commit and rerun/check Actions. Avoid flooding CI with needless commits.
5. Share a brief gate report including repository, PR, exact commit, checks and their statuses, and the GitHub Actions run and artifact URLs when present. State `passed`, `failed`, `pending`, `blocked` or `not run` precisely. **Do not claim any test was run when it was not.**
6. **Only after a green, relevant GitHub gate** give optional Windows/Pixel installation instructions (or a verified CI artifact path). Distinguish an APK that built on GitHub from an APK that is safe to `adb install -r`: GitHub runner debug signing may differ from the user's existing installation. Confirm signer and package identity before replacement; **never propose uninstall/data reset as a shortcut**.
7. Physical Pixel validation is a separate gate for microphone, NPU/Tensor, ASR/LLM/TTS, notification access, Android lifecycle, timing, RAM/thermal and real-world behavior. A green CI build does **not** prove these aspects. Preserve private audio, notification content, model weights and credentials outside Git.

## User not at a computer

Treat absence from a computer as a normal workflow. Continue reviewing, editing, pushing and building through GitHub without requiring the user to open Android Studio/PowerShell merely to discover a compile error. Do not promise background activity or later delivery; inspect current CI in this turn whenever reporting a build result. Ask for physical-device interaction only when GitHub cannot validate the specific behavior.

## Product focus

Mosaic is currently **voice-first and tool-first**. The first tool reads the latest accessible WhatsApp notification on-device with user-granted access and reads it aloud; no unrestricted WhatsApp history, automated sends, or remote Core dependency. Nutrition, photos and similar features are paused as primary UIs but retained for future optional tools.

## Current example (not a permanent acceptance claim)

Android [Draft PR #44](https://github.com/adarcook/mosaic-android/pull/44) is stacked on the experimental voice branch. Its GitHub Actions runs for its then-current commit `3c27ab79c4d9b2feea5fb6363f7e65dede87f2ad` reported success for the LLM, Zipformer voice and Android build workflows. This confirms a hosted compilation/test gate on that revision, **not** on-device WhatsApp functionality, installer signature compatibility, or future commits.
