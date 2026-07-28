# API and Event Contracts

All client-to-core interactions use versioned typed contracts. Canonical schemas should live in the shared contracts package and generate or validate Kotlin and Python models.

## Initial REST endpoints

```text
POST   /v1/sync/batches
GET    /v1/sync/checkpoints/{client_id}
POST   /v1/artifacts
GET    /v1/artifacts/{artifact_id}
POST   /v1/query
GET    /v1/query/{query_id}
POST   /v1/memories/candidates
PATCH  /v1/memories/{memory_id}
GET    /v1/sources
GET    /v1/health
```

## Sync requirements

- batches are idempotent;
- events have globally unique IDs;
- clients may retry safely;
- responses include per-item acceptance or validation errors;
- deletes use tombstones;
- schema evolution remains backward-compatible within a major version.

## Common event envelope

```json
{
  "event_id": "uuid",
  "event_type": "nutrition.meal.recorded",
  "event_version": 1,
  "occurred_at": "2026-07-28T12:00:00+03:00",
  "recorded_at": "2026-07-28T12:01:00+03:00",
  "producer": "mosaic-fit",
  "subject_id": "user-id",
  "correlation_id": "uuid",
  "payload": {},
  "source": {
    "device_id": "device-id",
    "local_record_id": "room-primary-key"
  }
}
```

## Initial domain events

Nutrition: meal recorded/updated/deleted, analysis completed and daily summary calculated.

Swimming: session recorded/updated, plan created and analysis completed.

Photos: asset indexed, face clustered and entity labelled.

## Query responses

Answers include confidence, citations with source/artifact locators, derived data, warnings and an execution ID. Internal worker events remain explicit and versioned, including artifact ingestion, extraction, embedding, entity extraction, index refresh and memory-candidate creation.