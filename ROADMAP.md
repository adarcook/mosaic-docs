# Mosaic Roadmap

## Phase 0 — Foundation

- Define repository boundaries and architecture
- Establish versioned shared contracts
- Create minimal project skeletons
- Document privacy and trust assumptions

## Phase 1 — Meal capture MVP

- Create the Android application shell
- Capture or select a meal photo
- Upload securely to Mosaic Server
- Run analysis through a replaceable adapter
- Return a validated meal-analysis response
- Let the user correct and confirm the result
- Store a local nutrition log

## Phase 2 — Reliable VPS operation

- Add authentication
- Add PostgreSQL persistence
- Add background jobs and retry behavior
- Add encrypted secrets and deployment configuration
- Define raw-image retention and deletion
- Add observability and backups

## Phase 3 — Fitness context

- Track weight and measurements
- Import or record workouts
- Import swimming sessions from supported Android health sources
- Build daily and weekly nutrition and training summaries

## Phase 4 — Mosaic Core synchronization

- Synchronize confirmed records from the VPS
- Index trusted records and their provenance
- Add source-aware retrieval
- Generate cross-domain insights with citations

## Phase 5 — Broader Mosaic ecosystem

- Integrate Mosaic Photos
- Add development and project context
- Add permission-aware tools and automations
- Expand to additional domain applications

## Immediate next milestone

Deliver an end-to-end vertical slice:

```text
Android photo
→ VPS upload
→ analyzer result
→ contract validation
→ user confirmation
→ saved meal record
```
