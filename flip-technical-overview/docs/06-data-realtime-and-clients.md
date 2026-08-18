# 06 — Data, Realtime, and Clients

## One product, several state-delivery mechanisms

Flip supports server-rendered web interaction and a React/TypeScript client architecture used for native desktop and mobile packaging. The difficult problem is not drawing the same screens twice; it is ensuring that every client observes one authoritative product state while still feeling immediate.

<img src="../diagrams/client-synchronization.svg" alt="Web and native realtime data flow" width="900" />

## Canonical data model

PostgreSQL remains authoritative for durable product state:

- accounts, sessions, and memberships;
- rooms, messages, replies, reactions, pins, and read state;
- forums, threads, replies, votes, bookmarks, and tags;
- synthesis runs, source relationships, feedback, and linkback;
- AI replies, citations, artifacts, and continuation state;
- notifications, settings, support/billing records, and audit events;
- Oban jobs and terminal workflow state.

Clients may cache or optimistically project these records. They do not become an alternate system of record.

## Three realtime paths

### 1. Electric synchronization

Electric shape streams are used for durable state that clients must replicate and recover:

- selected room/message views;
- forum threads and replies;
- settings and read models;
- records required for offline or reconnect behavior.

A shape is a server-authorized projection, not arbitrary database access. The client subscribes to the minimum relevant dataset and applies changes to its local state.

### 2. Phoenix channels and PubSub

Channels carry ephemeral or highly interactive events:

- typing;
- presence;
- targeted transient progress;
- low-latency notifications;
- specialized room/thread events that do not need durable replication.

A client that disconnects can tolerate losing a typing event. It cannot tolerate losing a committed message.

### 3. HTTP commands

Mutations with product semantics remain explicit server commands:

- send/edit/delete message;
- create/reply/vote/bookmark;
- update settings;
- invoke an AI or artifact workflow;
- perform moderation or support actions.

The server validates authorization and returns a transaction identity or durable result that client optimism can reconcile against.

## Optimistic mutation and reconciliation

A responsive client may render a pending local change before the server transaction arrives through synchronization.

```text
user action
  -> client creates optimistic item with local identity
  -> HTTP command reaches server
  -> server authorizes and commits
  -> response returns canonical identity / transaction marker
  -> Electric change arrives
  -> client reconciles optimistic and canonical record
```

Required behaviors:

- accepted mutations converge without duplicate rows;
- rejected mutations visibly roll back;
- reconnect does not reapply a completed command;
- client-generated identities cannot override server identity or ownership;
- permission changes invalidate stale local affordances;
- ordering remains stable under delayed sync.

## Data ownership

| Concern | Client | Server |
|---|---:|---:|
| Draft input and local UI state | Owns | Does not need |
| Optimistic placeholder | Temporary | Does not trust |
| Durable message/thread | Caches | **Owns** |
| Membership and authorization | Displays | **Owns** |
| Presence/typing | Emits/projected | Coordinates |
| AI job request | Requests | **Owns admission and lifecycle** |
| Citation/artifact identity | Renders | **Owns** |
| Feature configuration | Displays/caches | **Owns** |
| Sync shape authorization | Requests | **Owns** |

## Web client

Phoenix LiveView is suitable for server-driven product surfaces that benefit from:

- direct use of server contexts;
- minimal duplicated client domain logic;
- low-latency PubSub updates;
- administrative and operational interfaces;
- progressive enhancement through JavaScript hooks.

The web path can coexist with the React client. They are two projections over the same product contexts, not separate backends.

## React/TypeScript client

The shared client architecture provides:

- chat, forum, and AI/artifact views;
- typed API boundaries;
- Electric shape consumption;
- optimistic mutation/reconciliation;
- durable local state and endpoint configuration;
- platform adapters for desktop/mobile features.

The client should keep product-specific state transitions in shared data/application modules rather than coupling them to one screen or native wrapper.

## Desktop packaging

Tauri provides a native desktop shell around the client and can integrate:

- system tray and window lifecycle;
- native notifications;
- deep links;
- secure local preferences;
- updater and signed-release flow;
- platform file dialogs and selected OS capabilities.

The native shell does not bypass server authorization or turn the local cache into canonical state.

## Mobile packaging

Capacitor allows the shared React application to target iOS and Android while adding platform-specific bridges for notifications, deep links, secure storage, and lifecycle.

Shared code does not imply automatic parity. Mobile requires explicit handling of:

- suspended/background execution;
- intermittent connectivity;
- push token lifecycle;
- smaller memory/storage budgets;
- platform permission prompts;
- app-store signing and distribution;
- large artifact/media behavior.

## Offline semantics

“Offline-capable” must be defined per action.

| Operation | Reasonable offline behavior |
|---|---|
| Read previously synchronized content | Serve local cache with stale/offline indicator. |
| Draft a message | Retain locally. |
| Queue a mutation | Possible only with idempotent command identity and visible pending state. |
| Invoke AI research/media | Requires server/provider capacity; queue or reject honestly. |
| Change membership/moderation | Generally require online authoritative result. |
| Presence/typing | Disable while offline. |
| Open citation/artifact | Use cached metadata or state that source is unavailable. |

The architecture avoids claiming full local-first write authority where conflict semantics are undefined.

## Search

Product search combines server-side indexes with client presentation:

- chat and forum text indexes;
- scoped filters for community, room, author, time, tags, and content type;
- source/provenance navigation;
- optional semantic or AI-assisted discovery;
- pagination and stable ordering.

Client-local search can improve responsiveness over synchronized content but cannot reveal records outside server-authorized shapes.

## Notifications

A durable notification record and its delivery channel are separate:

- server creates the notification based on product state;
- web/native clients synchronize unread state;
- PubSub or push can alert a currently connected or backgrounded device;
- opening/reading updates durable state;
- duplicate delivery should not create duplicate notification records.

## Attachments and artifacts

Large media should not be duplicated through every realtime channel. The durable record carries metadata, ownership, lifecycle, and a storage/provider reference. Clients fetch/render the object through an authorized path and respond to lifecycle updates.

## Failure and recovery

| Failure | Client behavior |
|---|---|
| HTTP mutation rejected | Roll back optimistic state and show domain error. |
| Sync stream disconnects | Reconnect from durable cursor/shape state. |
| Channel disconnects | Restore ephemeral subscriptions; do not replay typing. |
| Duplicate server event | Deduplicate by canonical identity/transaction. |
| Local cache corrupt/stale | Rebuild from authorized server projection. |
| Permission revoked | Remove inaccessible records/affordances according to policy. |
| Artifact still pending | Render durable pending state, not a broken link. |
| Endpoint unavailable | Keep core local UI state and present explicit connectivity status. |

## Architectural constraint

Every new client feature must identify which state class it uses: durable transaction, synchronized projection, ephemeral event, or local-only UI state. Features that mix these implicitly are the most common source of duplicate messages, stale permissions, and reconnect bugs.
