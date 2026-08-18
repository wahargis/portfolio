# ADR 0002 — Separate synthetic technical environment

- **Status:** Accepted
- **Decision scope:** Public technical review

## Context

A public architecture scenario is useful only if it exercises the real product contracts. Giving reviewers production access or copying production data would create privacy and security risk.

## Decision

Maintain a separate synthetic technical environment that shares the product architecture and migration history but has independent:

- database and storage;
- credentials and secrets;
- sessions and administrative state;
- provider configuration;
- queues and realtime namespaces;
- synthetic fixtures.

Public scenario documentation describes expected product transitions without publishing reset credentials or operational secrets.

## Consequences

The environment requires fixture and deployment maintenance. It provides inspectable provenance, authorization, AI/tool, artifact, and synchronization scenarios without relying on private data.

## Revisit when

A fully local reproducible public distribution becomes preferable to a hosted synthetic environment.
