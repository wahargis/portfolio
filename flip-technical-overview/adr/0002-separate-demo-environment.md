# ADR 0002 — Separate synthetic technical environment

- **Status:** Accepted
- **Decision scope:** Public technical deployment

## Context

Flip's product contracts are best demonstrated through a running environment, but production accounts, messages, credentials, moderation authority, and operational state cannot be exposed for that purpose.

A static mock-up would not exercise authorization, workflow, AI, artifact, synchronization, and failure behavior. Reusing production data or authority would create unacceptable privacy and security risk.

## Decision

Maintain a separate synthetic technical environment that can run the product architecture and migration model while using independent:

- database and file storage;
- credentials and provider configuration;
- account sessions and administrative state;
- queues and realtime namespaces;
- generated communities, conversations, documents, sources, and artifacts;
- sanitized telemetry and failure fixtures.

The synthetic deployment has no route to production data or administration. Public documentation describes its product types, state transitions, and isolation boundary without publishing reset credentials or operational secrets.

## Consequences

The environment requires fixture, migration, and deployment maintenance. In return, product authorization, AI/tool execution, provenance, artifact lifecycle, conversation curation, client synchronization, and controlled failures can be represented through actual application state without relying on private data.

## Revisit when

Revisit if a fully local reproducible distribution can provide the same application behavior and isolation more effectively than the hosted synthetic environment.
