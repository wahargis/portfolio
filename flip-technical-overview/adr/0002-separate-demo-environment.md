# ADR 0002: Separate synthetic technical environment

- **Status:** Accepted
- **Scope:** Public scenarios, deterministic tests, and technical review

## Context

Flip's product deployment contains accounts, community content, credentials, provider configuration, queues, storage, telemetry, and administrative authority that should not be copied into a public technical environment.

The architecture still needs repeatable scenarios for authorization, agent execution, tools, curation, asynchronous artifacts, client synchronization, and failure recovery.

## Decision

Run public scenarios in a separate synthetic environment with its own:

- database and synthetic identities;
- queues and worker namespace;
- object storage and artifact roots;
- credentials and provider fixtures or independently scoped keys;
- callback and webhook routes;
- hostnames, deep links, and client capability configuration;
- telemetry and exported evidence.

The synthetic environment uses the same kinds of application contracts and state transitions but does not proxy unknown requests to product resources.

## Consequences

Scenario fixtures and assertions require maintenance as product contracts change. Live-provider evaluation remains separate from deterministic fixtures. The environment cannot establish product scale or availability.

It can provide inspectable evidence for execution, authorization, persistence, retry, continuation, and client convergence without product data or authority.

## Revision conditions

Reconsider only if another environment can provide equivalent contract coverage with equal or stronger separation from product resources.
