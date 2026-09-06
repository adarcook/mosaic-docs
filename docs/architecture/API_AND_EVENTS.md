# API and Event Contracts

All cross-component business records and synchronization events use versioned typed contracts. Canonical schemas live in `mosaic-contracts` and should generate or validate Kotlin and Python models.

The primary Android-to-Core transport is asynchronous synchronization through Firebase, not a required direct REST connection. Direct HTTP endpoints may still exist for local development, optional interactive access or narrowly scoped services, but Android meal recording must not depend on Core reachability.

## Optional and local REST endpoints

Potential local or service endpoints include:

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

These endpoints are not the required Phase 1 mobile synchronization path. The accepted mobile path is Room/outbox → Firestore → Core consumer.

## Sync requirements

- batches or event publications are idempotent;
- events have globally unique IDs;
- clients may retry safely;
- acceptance and validation failures are explicit;
- corrections and business deletion semantics preserve history through new immutable events or tombstones where defined;
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

The exact wire structure remains owned by `mosaic-contracts` even when examples in this document use simplified field names.

## Meal capture and analysis boundary

Meal recording and meal analysis are separate concerns.

```text
Manual entry ───────────────┐
                            ↓
                     user-confirmed meal
                            ↓
                  canonical MealRecord
                            ↓
                 immutable meal event

Camera → MealPhotoInput → MealAnalyzer
                         ├─ on-device adapter (preferred production path)
                         └─ remote/HTTP adapter (development or explicit fallback)
                                  ↓
                           analysis estimate
                                  ↓
                         user review/correction
                                  ↓
                     canonical MealRecord
```

`MealAnalyzer` is an Android implementation boundary, not itself a canonical wire contract. Its concrete runtime may change without changing the meal-record contract or capture UI.

Rules for analyzer output:

- an analysis result is an estimate and is not a trusted meal fact by itself;
- assumptions, confidence and original estimated values should be retained when available;
- the user may correct the estimate before confirmation;
- only confirmed structured output becomes a canonical `MealRecord` revision;
- meal-image bytes are not included in ordinary synchronization events by default;
- Core availability is not required for immediate image-assisted capture on a capable device.

The current Android implementation is transitioning from a legacy `MealAnalysis`-centric local storage model toward the canonical `MealRecord`; event-outbox work must use the canonical model rather than cementing the compatibility representation.

## Initial domain events

Nutrition:

- `nutrition.meal.recorded`;
- `nutrition.meal.updated`;
- `nutrition.meal.deleted`;
- `nutrition.analysis.completed`;
- `nutrition.daily-summary.calculated`.

`nutrition.analysis.completed` is optional provenance for an analysis result when there is a reason to synchronize that result. It is not a substitute for a confirmed meal record and is not required for local on-device analysis.

Inventory:

- `inventory.item.created`;
- `inventory.item.updated`;
- `inventory.stock.adjusted`;
- `inventory.purchase.recorded`;
- `inventory.consumption.requested`;
- `inventory.consumption.confirmed`;
- `inventory.match.corrected`.

Swimming:

- `swimming.session.recorded`;
- `swimming.session.updated`;
- `swimming.plan.created`;
- `swimming.analysis.completed`.

Photos:

- `photos.asset.indexed`;
- `photos.face.clustered`;
- `photos.entity.labelled`.

## Meal-component inventory link

A meal component may carry an optional Inventory reference. The nutrition record remains valid when the reference is absent.

```json
{
  "meal_component_id": "uuid",
  "food_id": "food-chicken-breast",
  "display_name": "Chicken breast",
  "quantity": 200,
  "unit": "g",
  "preparation_state": "cooked",
  "inventory_match": {
    "inventory_item_id": "uuid",
    "status": "user_confirmed",
    "confidence": 1.0,
    "matched_at": "2026-07-28T12:02:00+03:00"
  }
}
```

The link must preserve whether the match was model-suggested, rule-based or confirmed by the user. Correcting a match creates a new version instead of erasing the previous association.

## Inventory consumption request

Mosaic Fit or Core may publish a request, but Mosaic Inventory owns the final stock decision.

```json
{
  "event_type": "inventory.consumption.requested",
  "event_version": 1,
  "payload": {
    "meal_id": "uuid",
    "meal_component_id": "uuid",
    "inventory_item_id": "uuid",
    "consumed_quantity": 200,
    "consumed_unit": "g",
    "preparation_state": "cooked",
    "requires_conversion": true,
    "match_status": "user_confirmed"
  }
}
```

Inventory may accept, reject or request confirmation. A confirmed stock adjustment must reference both the request event and the Inventory ledger entry. Raw-to-cooked conversions, shared portions and uncertain matches must not be silently resolved by Fit or Core.

## Query and Insight evidence

Answers and Insights include confidence where applicable, citations or evidence references with source/artifact locators, derived data, warnings and an execution or analysis-run identifier.

Internal worker events remain explicit and versioned, including artifact ingestion, extraction, embedding, entity extraction, index refresh and memory-candidate creation.