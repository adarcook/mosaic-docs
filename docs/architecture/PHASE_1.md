# Phase 1 Plan

## Goal

Deliver one end-to-end vertical slice that proves Mosaic can receive a trusted domain record, preserve provenance, index it, retrieve it and answer a cited cross-domain question.

The first slice uses Mosaic Fit because nutrition records are already relevant, structured and easy to review manually.

## Scope

1. Define versioned meal and sync contracts.
2. Record or confirm a meal in the Android client.
3. Store the operational record locally in Room.
4. Synchronize an idempotent event batch to Core.
5. Persist a normalized projection and source reference in PostgreSQL.
6. Create retrieval text and an embedding.
7. Query the record through the Mosaic API.
8. Return an answer with a resolvable citation.
9. Support user correction without erasing the original estimate.

## Milestones

### Foundation

- monorepo skeleton and dependency boundaries;
- local Docker Compose stack;
- database migrations;
- shared contract validation;
- health and readiness endpoints.

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

## Deferred

Wear OS ingestion, full photo-library indexing, autonomous agents, public internet exposure, multi-user tenancy and complex distributed queues remain outside Phase 1.