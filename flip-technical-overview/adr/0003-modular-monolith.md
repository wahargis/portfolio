# ADR 0003: Modular Phoenix application

- **Status:** Accepted
- **Scope:** Main product deployment

## Context

Flip's main workflows cross identity, membership, chat, forums, search, AI activity, curation, media, background work, and client delivery. Several operations require shared transactions or current authorization across these domains.

Splitting the application into independently deployed services would add network failure, distributed transactions, duplicated authorization, and more operational state before those boundaries are justified by measured load or team ownership.

## Decision

Keep the primary product as one Phoenix and Ecto application with explicit contexts and service boundaries. Use PostgreSQL as shared durable authority, Oban and supervised processes for background isolation, and provider adapters for external services.

Contexts own their schemas and state transitions. Agent tools call the owning product context rather than writing across domains directly.

## Consequences

The repository and application are large and require disciplined context ownership. Expensive background work must be isolated by queues, processes, and concurrency limits. Internal boundaries need tests because deployment does not enforce them.

The application retains straightforward product transactions and consistent authorization across chat, forums, curation, AI effects, and client state.

## Revision conditions

Extract a workload when it has a stable contract, independent operating requirements, demonstrated scaling or fault-isolation need, and a clear solution for authorization and transaction boundaries.
