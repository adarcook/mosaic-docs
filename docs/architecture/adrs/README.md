# Architecture Decision Records

## ADR 0001 — Multi-repository architecture

**Status:** Accepted

Keep Mosaic in separate repositories aligned with independently deployed runtimes and ownership boundaries. The current repositories are `mosaic-core`, `mosaic-server`, `mosaic-android`, `mosaic-contracts` and `mosaic-docs`. Cross-repository compatibility is managed through versioned contracts rather than shared implementation packages.

A monorepo is not the current target. It may be reconsidered only if coordinated changes and duplicated tooling become a persistent, measurable problem.

## ADR 0002 — Domain applications own operational data

**Status:** Accepted

Mosaic Fit, Mosaic Swim and Mosaic Photos remain independent domain applications. They own their workflows and operational records; Mosaic Core stores normalized projections, references and cross-domain knowledge.

## ADR 0003 — Local-first data with hybrid compute

**Status:** Accepted

Canonical personal data stays on the home node while scoped jobs may run on a VPS or approved cloud model. Remote payloads must be minimal, policy-controlled, temporary and auditable.

## ADR 0004 — PostgreSQL and pgvector for Phase 1

**Status:** Proposed

Use PostgreSQL as the primary Core database and pgvector as the initial vector index. This keeps transactions, relational queries, full-text search, metadata filtering and vector retrieval in one operational system.
