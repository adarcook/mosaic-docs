# Contributor / AI coding agent instructions — Mosaic Docs

GitHub `main`, open PRs and [ROADMAP.md](ROADMAP.md) are the authoritative project state. Read them before any development work. A draft PR is not a completed roadmap stage.

**Mandatory new-version gate:** Follow [docs/DEVELOPMENT_WORKFLOW.md](docs/DEVELOPMENT_WORKFLOW.md) for every implementation or APK version: push the change to a Draft PR, let appropriate GitHub Actions build and run tests on the **latest head SHA**, inspect the run results and fix failed builds **before** telling the user to compile or install locally. Where GitHub CI cannot test a hardware-specific requirement, disclose that gap. Do not claim success from an older SHA, a queued job or an unexecuted test.

When the user is away from a computer, do not require local build attempts just to discover compile problems. Physical Android acceptance, signing compatibility and permissions remain separate checks.

Keep work in small reviewable slices. Never merge without explicit user permission. Never commit personal notification/audio content, model weights, secrets or signing credentials. The current product focus is the on-device Hebrew voice agent and permission-scoped tools; nutrition and photos are optional future tool domains.
