# Architecture Decision Records

## ADR 0001 — Monorepo

**Status:** Accepted

Maintain Mosaic Core, shared contracts and domain clients in one monorepo with strict logical boundaries. This reduces contract drift and supports coordinated Python/Kotlin changes while preserving independent deployment.

## ADR 0002 — Domain applications own operational data

**Status:** Accepted

Mosaic Fit, Mosaic Swim and Mosaic Photos remain independent domain applications. They own their workflows and operational records; Mosaic Core stores normalized projections, references and cross-domain knowledge.

## ADR 0003 — Local-first data with hybrid compute

**Status:** Accepted

Canonical personal data stays on the home node while scoped jobs may run on a VPS or approved cloud model. Remote payloads must be minimal, policy-controlled, temporary and auditable.

## ADR 0004 — PostgreSQL and pgvector for Phase 1

**Status:** Proposed

Use PostgreSQL as the primary Core database and pgvector as the initial vector index. This keeps transactions, relational queries, full-text search, metadata filtering and vector retrieval in one operational system.