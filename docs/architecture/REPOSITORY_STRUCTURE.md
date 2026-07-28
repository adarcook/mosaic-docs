# Repository Structure

## Recommended monorepo

```text
mosaic/
├── apps/
│   ├── core-api/              # FastAPI gateway and query API
│   ├── core-worker/           # ingestion, enrichment and scheduled jobs
│   ├── console/               # local web/desktop management UI
│   ├── fit-android/           # nutrition and fitness client
│   ├── swim-android/          # phone application
│   ├── swim-wear/             # Wear OS application
│   └── photos/                # local photo search UI and CLI
├── packages/
│   ├── contracts/             # versioned API/event schemas
│   ├── domain-model/          # shared conceptual model
│   ├── python-sdk/            # generated or handwritten Python client
│   ├── kotlin-sdk/            # generated Kotlin models/client
│   ├── model-gateway/         # local, VPS and cloud model adapters
│   ├── retrieval/             # hybrid search interfaces
│   ├── policy/                # permissions and data classification
│   └── observability/         # logs, traces and audit helpers
├── services/
│   ├── postgres/
│   ├── object-store/
│   └── queue/
├── infra/
│   ├── compose/
│   ├── home-node/
│   └── vps/
├── docs/
│   ├── architecture/
│   ├── adrs/
│   └── operations/
└── tools/
    ├── migrations/
    ├── codegen/
    └── development/
```

## Dependency rules

- Domain applications may depend on contracts and generated SDKs, not on Core implementation packages.
- Core services may depend on domain-neutral packages but not Android UI modules.
- Domain-specific analysis stays inside its domain package unless it is genuinely reusable.
- Model providers are accessed only through the model gateway.
- Persistence implementations sit behind repository interfaces.
- Cross-domain reads happen through Mosaic Core projections and source references.

## Why a monorepo

The project is primarily developed by one person and coordinated changes across Python, Kotlin, schemas and documentation are frequent. A monorepo reduces contract drift, enables end-to-end validation and still permits independent deployment.

## Build and CI

CI should detect changed paths and run only relevant checks while always validating shared contracts. Recommended checks include schema compatibility, Python tests and linting, Android unit tests, migration validation, Mermaid/Markdown link checks and container builds for changed services.

## Migration path

The existing repositories can be imported gradually with preserved history. During transition, `mosaic-contracts` remains the source of truth until its schemas move under `packages/contracts`; applications can continue releasing independently.