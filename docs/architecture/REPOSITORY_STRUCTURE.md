# Repository Structure

## Recommended multi-repository structure

Mosaic should continue using separate repositories for independently deployed platforms and services. The repository boundary reflects ownership, runtime, release cadence and technology rather than forcing all code into one build system.

```text
adarcook/
├── mosaic-core/         # Local personal-intelligence layer on the home computer
├── mosaic-server/       # VPS-facing APIs, remote jobs and synchronization services
├── mosaic-android/      # Android domain application and future Wear OS modules
├── mosaic-contracts/    # Versioned API schemas, event schemas and generated models
└── mosaic-docs/         # Architecture, ADRs, roadmap and operational documentation
```

Additional domain repositories can be added when they become real deployable products, for example `mosaic-photos` or a dedicated swimming application. They should not be split merely to create conceptual boundaries; a new repository is justified by an independent runtime, release lifecycle or substantial domain ownership.

## Repository responsibilities

### `mosaic-core`

- Runs primarily on the home computer.
- Owns ingestion, indexing, retrieval, unified memory and cross-domain reasoning.
- Stores normalized projections and provenance without taking over domain workflows.
- Exposes local query, administration and synchronization interfaces.

### `mosaic-server`

- Runs on the VPS.
- Handles authenticated remote access, temporary compute jobs and synchronization when the home node is unavailable.
- Must not become the canonical store for unrestricted personal data.
- Communicates through contracts published by `mosaic-contracts`.

### `mosaic-android`

- Owns Android user workflows such as nutrition capture, fitness records and device integration.
- Keeps operational domain data locally where appropriate.
- Sends confirmed records or events rather than exposing its database directly.
- May contain multiple Gradle modules for phone, Wear OS and shared Android code.

### `mosaic-contracts`

- Is the source of truth for API and event compatibility.
- Contains versioned OpenAPI, JSON Schema or Protocol Buffer definitions.
- Publishes or generates Kotlin and Python models when useful.
- Defines compatibility rules and deprecation policy.

### `mosaic-docs`

- Holds architecture documents, ADRs, diagrams, roadmap and operational runbooks.
- Documents decisions that affect more than one repository.
- Links implementation work across repositories without owning executable code.

## Dependency rules

- Domain applications depend on versioned contracts, never on Mosaic Core implementation code.
- Mosaic Core and Mosaic Server do not import Android modules.
- Cross-repository communication uses documented APIs and events rather than shared database access.
- Domain-specific analysis remains in its owning repository unless it becomes a stable, reusable service.
- Model providers are accessed through explicit adapters in the repository that executes the request.
- Every synchronized record carries source, version, timestamps and provenance.

## Coordinating changes across repositories

A contract change normally follows this order:

1. Propose and review the schema change in `mosaic-contracts`.
2. Publish a backward-compatible contract version or generated models.
3. Update consumers such as `mosaic-server`, `mosaic-core` and `mosaic-android` in separate PRs.
4. Deploy consumers before removing deprecated fields or event versions.
5. Record major cross-repository decisions in `mosaic-docs`.

For early development, contracts may be consumed by Git commit or local checkout. Once releases stabilize, use tagged versions and automated dependency updates.

## CI expectations

Each repository owns its own tests and deployment pipeline. In addition:

- `mosaic-contracts` validates schemas and backward compatibility.
- Consumer repositories run contract tests against their pinned contract version.
- End-to-end tests may use a small orchestration repository or CI workflow, but production code remains in its owning repository.
- Documentation checks validate Markdown links and Mermaid diagrams in `mosaic-docs`.

## Why multi-repo fits Mosaic now

The existing repositories already map cleanly to separate runtimes: home computer, VPS, Android and shared contracts. Keeping these boundaries avoids a disruptive migration, allows independent releases and makes privacy and deployment responsibilities explicit. A monorepo can be reconsidered later only if coordinated changes and duplicated tooling become a persistent cost that outweighs these benefits.
