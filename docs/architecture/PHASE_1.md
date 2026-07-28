# Phase 1 Plan

## Goal

Deliver one end-to-end vertical slice that proves Mosaic can receive a trusted domain record, preserve provenance, index it, retrieve it and answer a cited cross-domain question.

The first slice uses Mosaic Fit because nutrition records are already relevant, structured and easy to review manually. The meal model must also be designed so it can later connect to Mosaic Inventory without replacing or reshaping the core nutrition records.

## Scope

1. Define versioned meal and sync contracts.
2. Record or confirm a meal in the Android client.
3. Store the operational record locally in Room.
4. Represent each meal as structured food components with quantity, unit and preparation state when known.
5. Allow an optional reference from a meal component to an inventory item, while keeping nutrition records valid when no inventory match exists.
6. Synchronize an idempotent event batch to Core.
7. Persist a normalized projection and source reference in PostgreSQL.
8. Create retrieval text and an embedding.
9. Query the record through the Mosaic API.
10. Return an answer with a resolvable citation.
11. Support user correction without erasing the original estimate or earlier inventory match.
12. Define, but do not yet fully automate, a versioned inventory-consumption event contract.

## Inventory readiness

Phase 1 does not require a complete Inventory application or automatic stock deduction. It must establish the integration boundary so that a future Inventory domain can consume confirmed meal data safely.

The meal contract should support:

- stable meal-component IDs;
- normalized food identity where available;
- quantity and unit;
- preparation state such as raw, cooked or unknown;
- an optional `inventory_item_id` or external item reference;
- match confidence and whether the match was suggested or user-confirmed;
- correction history and provenance;
- an event such as `inventory.consumption.requested` that Inventory may process later.

Inventory remains the owner of stock levels, purchases, expiry dates, unit conversions and final deduction decisions. Mosaic Fit must not directly modify the Inventory database.

## Milestones

### Foundation

- establish the existing multi-repository boundaries and local development integration;
- local Docker Compose stack for Core dependencies;
- database migrations;
- shared contract validation across `mosaic-contracts`, Android and Python services;
- health and readiness endpoints.

### Meal structure and inventory readiness

- structured meal components;
- optional inventory-item references;
- preparation-state and unit fields;
- user-confirmed matching and correction history;
- versioned inventory-consumption event schema.

### Synchronization

- device identity;
- checkpoints and idempotency keys;
- event validation;
- retry-safe client queue;
- tombstone handling.

### Retrieval and provenance

- source, artifact, event and evidence tables;
- full-text and vector search;
- citation resolver;
- basic query endpoint.

### Memory

- candidate memory creation;
- approval/rejection workflow;
- versioned corrections;
- confidence and provenance display.

## Acceptance criteria

A confirmed meal created on Android can be synchronized twice without duplication, retrieved after restart, corrected while retaining history, and used in a query whose answer links back to the exact source record.

Each meal contains structured components that remain useful without Inventory, but can optionally reference an Inventory item. The contracts clearly distinguish cooked meal quantities from raw stock quantities, retain match provenance and expose a versioned consumption-request event without automatically deducting stock.

## Deferred

A complete Inventory user interface, automatic stock deduction, recipe-level stock conversion, Wear OS ingestion, full photo-library indexing, autonomous agents, public internet exposure, multi-user tenancy and complex distributed queues remain outside Phase 1.