# ADR 0003 — Modular Phoenix monolith

- **Status:** Accepted
- **Decision scope:** Product and data architecture

## Context

Chat, forum, synthesis, AI replies, citations, artifacts, notifications, and authorization share users, communities, transactions, realtime updates, and relational provenance. Splitting them into services would add network failure and eventual consistency before a demonstrated scale or ownership need.

## Decision

Use one Phoenix application with explicit domain contexts, one PostgreSQL authority, one migration path, one PubSub system, and Oban-backed asynchronous workflows.

Contexts communicate through public domain functions and durable identifiers. “Single application” does not permit arbitrary cross-context table mutation.

## Consequences

Cross-domain transactions and source relationships remain straightforward. The application can be deployed compactly and scaled by nodes/roles before service extraction.

The cost is boundary discipline: large contexts or workers must be decomposed internally, and tests must detect unauthorized cross-domain coupling.

## Revisit when

A domain has independently demonstrated scaling, release, regulatory, or team-ownership requirements that outweigh distributed consistency cost.
