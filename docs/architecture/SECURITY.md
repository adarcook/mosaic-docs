# Security and Permissions

## Trust boundaries

Mosaic treats the home node, Android clients, VPS workers, cloud model providers and external connectors as separate trust zones.

## Data classification

Each source and artifact is classified as private, sensitive or restricted. Policy checks consider user identity, device identity, requested operation, destination model, connector scope and data classification.

## Core controls

- least-privilege connector scopes;
- per-device authentication and revocation;
- encryption in transit and at rest;
- explicit approval for destructive or externally visible actions;
- immutable audit records for tool calls, remote jobs and memory changes;
- scoped remote-job payloads with expiry;
- source-aware authorization before retrieval;
- secrets outside source control.

## Model routing

Sensitive data defaults to local processing. Remote processing is permitted only through an explicit policy that minimizes payloads and records the provider, purpose, model and retention assumptions.

## Memory safety

Model-generated memories begin as candidates unless produced by a trusted deterministic rule. User corrections take precedence, contradictions create new versions rather than silent overwrites, and every active memory remains traceable to evidence.

## Remote workers

VPS jobs receive only the required input, use short-lived credentials, store temporary encrypted data, and delete it according to an auditable retention policy. The VPS never receives broad filesystem access to the home node.

## Secrets

Use OS keychains, Docker secrets or a dedicated secrets manager. Maintain separate credentials per connector and environment, and support independent rotation and revocation.