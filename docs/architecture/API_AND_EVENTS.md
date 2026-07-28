# API and Event Contracts

All client-to-core interactions use versioned typed contracts. Canonical schemas should live in `mosaic-contracts` and generate or validate Kotlin and Python models.

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

Nutrition:

- `nutrition.meal.recorded`;
- `nutrition.meal.updated`;
- `nutrition.meal.deleted`;
- `nutrition.analysis.completed`;
- `nutrition.daily-summary.calculated`.

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

## Query responses

Answers include confidence, citations with source/artifact locators, derived data, warnings and an execution ID. Internal worker events remain explicit and versioned, including artifact ingestion, extraction, embedding, entity extraction, index refresh and memory-candidate creation.