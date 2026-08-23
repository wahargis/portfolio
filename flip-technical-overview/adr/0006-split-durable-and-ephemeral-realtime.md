# ADR 0006: Separate durable, asynchronous, realtime, and local state

- **Status:** Accepted
- **Scope:** Web and native-client data flow

## Context

Messages, forum items, AI activities, media jobs, typing indicators, and client drafts have different persistence and replay requirements. Treating all of them as realtime events would make product state unrecoverable. Persisting all transient interaction state would add unnecessary storage and synchronization.

## Decision

Use four state classes:

- PostgreSQL for durable product state;
- PostgreSQL plus Oban and supervised workers for durable asynchronous state;
- Phoenix Channels and PubSub for ephemeral realtime state;
- client memory or local storage for drafts and other local interaction state.

Server commands authorize and commit durable changes. Native clients receive authorized durable projections and reconcile optimistic state by local transaction and canonical object identity.

## Consequences

Clients must handle command responses, durable synchronization, and realtime events arriving in different orders. Access revocation must remove durable projections and local affordances.

The application can recover committed product and workflow state after disconnect or restart without replaying every transient event.

## Revision conditions

Change the owner or delivery path of a state class only when its durability, replay, consistency, or privacy requirements change and the replacement defines a clear source of truth.
