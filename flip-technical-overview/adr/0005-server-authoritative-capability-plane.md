# ADR 0005 — Server-authoritative capability plane

- **Status:** Accepted
- **Decision scope:** AI tools and product effects

## Context

A prompt can ask a model to respect permissions, but it cannot enforce them. Tool schemas, internal retrieval, and effectful product actions need the same authorization guarantees as ordinary user commands.

## Decision

The server computes the tool catalog from surface, feature state, provider availability, actor/community scope, and object context. Tool calls are parsed and dispatched through known handlers. Trusted scope is supplied by the runtime, not model arguments.

Internal retrieval fails closed without scope. Effect tools call domain services and return durable receipts/artifact identities.

## Consequences

Capability logic is more complex and requires parity tests. The model cannot expand its own permissions, perform arbitrary database actions, or convert missing scope into global search.

## Revisit when

A capability protocol can preserve the same server-side authorization and effect guarantees with less bespoke catalog logic.
