# ADR 0006 — Split durable and ephemeral realtime state

- **Status:** Accepted
- **Decision scope:** Client synchronization

## Context

Messages, threads, votes, and settings must recover after disconnect and support reconciliation. Typing, presence, and transient progress are valuable only in the moment. One transport and persistence policy is poorly suited to both.

## Decision

Use:

- PostgreSQL as canonical durable state;
- Electric shape synchronization for recoverable client projections;
- HTTP/domain commands for authoritative mutations;
- Phoenix channels/PubSub for ephemeral or targeted low-latency events.

Clients may render optimistic state but reconcile to canonical identities and transactions.

## Consequences

Clients coordinate multiple realtime paths and need explicit reconnect tests. Durable state is recoverable without persisting every ephemeral event.

## Revisit when

A replacement synchronization protocol can express both classes without weakening durability or adding unnecessary persistence.
