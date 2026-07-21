# Mosaic Architecture

## System overview

```text
Mosaic Fit (Android)
        |
        | HTTPS, versioned contracts
        v
Mosaic Server (VPS, always on)
        |
        | synchronization
        v
Mosaic Core (home computer, intermittently available)
```

## Component roles

### Mosaic Fit

Captures and presents domain data. It remains useful offline, stores local records in Room, uploads meal images when connectivity is available, and asks the user to confirm uncertain analysis results.

### Mosaic Server

Provides the online boundary. It authenticates clients, accepts uploads, invokes a replaceable analysis adapter, validates contract payloads, stores operational data, and queues synchronization for Mosaic Core.

### Mosaic Core

Provides durable intelligence. It indexes approved records and sources, manages long-term memory, retrieves evidence, enforces permissions, and generates cross-domain insights with citations.

### Mosaic Contracts

Defines implementation-neutral, versioned payloads. JSON Schema is initially canonical. Kotlin and Python bindings may later be generated from these definitions.

## Data lifecycle: meal photo

1. Mosaic Fit captures a photo and optional user notes.
2. The app uploads the image to Mosaic Server.
3. The server invokes the configured meal analyzer.
4. The analyzer returns ingredients, estimates, confidence, assumptions, and questions.
5. The result is validated against the shared contract.
6. Mosaic Fit presents the estimate for correction and confirmation.
7. Only the confirmed record becomes trusted nutrition data.
8. Mosaic Core synchronizes and indexes the confirmed record when available.

## Trust boundaries

- Images and personal records are private data.
- Secrets never belong in Git.
- The VPS should retain raw images only as long as operationally necessary.
- Model output is an estimate, not a trusted fact, until confirmed.
- Mosaic Core must retain provenance for imported and inferred data.

## Deployment direction

- Android application: user device
- Mosaic Server: Ubuntu VPS using containers
- Mosaic Core: home computer, local services and storage
- PostgreSQL: server operational data
- Room/SQLite: mobile local data

## Open decisions

- Authentication method between Android and VPS
- Encrypted storage and image-retention policy
- Synchronization conflict strategy
- Initial meal-analysis adapter implementation
- Contract generation strategy for Kotlin and Python
