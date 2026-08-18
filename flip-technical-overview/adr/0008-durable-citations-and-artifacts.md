# ADR 0008 — Durable citations and artifacts

- **Status:** Accepted
- **Decision scope:** Evidence and generated output

## Context

A citation token, chart, image, document analysis, or video embedded only in model prose has no reliable lifecycle, validation, permission, or retry identity.

## Decision

Represent citations, source records, artifacts, provider requests, and asynchronous terminal state as durable product objects. The model receives stable identities and refers to them in the final reply. The renderer resolves only valid, visible objects.

## Consequences

The product can validate evidence, deduplicate effects, show pending/failed state, continue asynchronous workflows, and apply retention policy. Additional schemas, cleanup, storage, and UI are required.

## Revisit when

No expected reversal. Specific storage formats and provider abstractions may change.
