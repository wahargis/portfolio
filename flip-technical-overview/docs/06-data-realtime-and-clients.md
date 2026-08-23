# Data, realtime, and clients

Flip serves server-rendered web views and React-based desktop and mobile clients. The clients share product semantics but receive state through several delivery paths. The architecture separates durable product records, durable background state, ephemeral realtime events, and local client interaction state.

![Web and native client synchronization](../diagrams/client-synchronization.svg)

## State classes

| State class | Examples | Authority | Delivery and recovery |
|---|---|---|---|
| **Durable product state** | Accounts, membership, rooms, messages, replies, forums, posts, reactions, polls, AI replies, citations, and artifacts. | PostgreSQL through product contexts. | Server-rendered views, APIs, durable synchronization, and reload from canonical records. |
| **Durable asynchronous state** | Jobs, runs, provider attempts, media progress, curation stages, retries, terminal failures, and continuations. | PostgreSQL with Oban and supervised workers. | Lifecycle queries, durable sync, targeted realtime updates, and restart from persisted state. |
| **Ephemeral realtime state** | Presence, typing, transient progress, local attention, and short-lived channel notifications. | Current application processes and channel state. | Phoenix Channels and PubSub; recreated or discarded after disconnect. |
| **Local interaction state** | Drafts, open views, pending local transactions, provisional attachments, and UI selection. | The client until a server command is accepted. | Local memory or device storage; reconciled against server response and durable sync. |

No single transport is used as the source of truth for all four classes.

## Server-authoritative commands

A client command includes authenticated session state, target object, command data, and a local transaction identity where optimistic reconciliation is required.

The server:

1. authenticates the current account and device session;
2. resolves current membership and target visibility;
3. validates command data and product state;
4. commits the canonical object or refusal;
5. records the transaction relationship where needed;
6. publishes durable and realtime updates;
7. returns the canonical result or structured error.

The client cannot make an optimistic object authoritative by assigning a local identifier. The server supplies the canonical identity and final lifecycle state.

## Durable synchronization

Durable synchronization projects authorized server records to the native client. The projection can include product objects, updates, deletions, and access-driven removal.

The sync boundary must account for:

- current account and community membership;
- object visibility and origin restrictions;
- server-side filters by product type and client capability;
- stable identifiers and update ordering;
- deletion, archival, and revocation state;
- schema and protocol version compatibility.

A client that reconnects should be able to rebuild durable state without replaying every realtime event that occurred while it was offline.

## Realtime channels

Channels carry low-latency events that are useful while connected:

- new or updated message notifications;
- presence and typing;
- transient tool or artifact progress;
- reaction and room activity;
- client attention and local interaction signals.

A realtime event can point to a durable object but does not replace it. When an event arrives before the corresponding durable record, the client can show a provisional update and reconcile after sync. When it arrives after sync, the client deduplicates by canonical identity and version.

## Optimistic reconciliation

Optimistic UI is useful for messages, reactions, uploads, and other interactive commands. It must handle several orderings:

```text
local optimistic object
  -> command response
  -> durable synchronization
  -> realtime event
```

or:

```text
local optimistic object
  -> realtime event
  -> durable synchronization
  -> delayed command response
```

The client uses local transaction identity, canonical object identity, and server version to collapse these paths into one visible object. A rejected command removes or marks the optimistic object and preserves an actionable error.

## AI and asynchronous state in clients

AI work can pass through pending, active, waiting, completed, failed, cancelled, and continued states. Clients should render the durable lifecycle rather than infer success from a streaming connection.

Examples include:

- an AI reply activity that has started but not yet committed a message;
- a generated image with provider progress and a pending artifact record;
- a curation run that created forum state but still needs linkback;
- a document operation that failed extraction before model analysis;
- a continuation scheduled after a video or file became available.

Transient progress can arrive over realtime channels. Reloaded clients obtain the current state from durable records.

## Capability negotiation

Web, desktop, mobile, and different deployments may not expose identical features. The server publishes capability metadata for the current account, deployment, and client protocol.

Capabilities can describe:

- authentication and session methods;
- push notifications and deep links;
- uploads and media constraints;
- reactions, emoji, and realtime features;
- durable synchronization and schema version;
- AI identities, tools, and multimodal operations available on a surface;
- route or provider-backed features that are currently configured.

Clients use this metadata to render only supported actions. A missing provider key or disabled connector should not appear as a normal control that fails after use.

## Access changes and local data

Membership or visibility can change while a client is connected or offline. The server applies current authorization to commands, reads, search, retrieval, and sync projections.

When access is revoked, the client must remove affected durable records and local affordances. A cached object or prior realtime event does not preserve authority. Sensitive data should not remain available through local search, previews, or background tasks after the server has removed it from the authorized projection.

## Error behavior

Client-visible errors use structured categories where possible:

- authentication or expired session;
- missing membership or forbidden target;
- stale object or version conflict;
- invalid command data;
- unavailable capability or provider;
- request limit or queue saturation;
- asynchronous operation failure;
- sync or protocol incompatibility;
- transient server or network failure.

The client can retry transient errors, request a refresh for stale state, or stop and explain deterministic refusal. It should not repeat an effectful command without the original transaction identity when the server may already have committed it.

## Testing requirements

Client and server tests cover:

- command response, sync, and realtime events arriving in different orders;
- duplicate delivery and repeated reconnect;
- optimistic success, rejection, and timeout;
- server restart during AI or artifact work;
- access revocation and local-data removal;
- client schema or capability mismatch;
- background completion while the client is offline;
- multiple clients acting on the same object;
- curation or media workflows with partial terminal state.

[Previous: Curation and provenance](05-synthesis-and-provenance.md) · [Next: Model routing and inference](07-model-routing-and-inference.md)
