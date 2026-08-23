# Flip System and Data Architecture

Flip is implemented as a modular Phoenix application backed by PostgreSQL. Product domains own their state and rules, while shared authorization, workflow, AI-activity, capability, and synchronization layers connect behavior that crosses those domains.

## Application boundaries

| Boundary | Responsibility |
|---|---|
| **Accounts and communities** | Users, sessions, profiles, membership, roles, community configuration, and actor identity. |
| **Chat** | Rooms, messages, replies, reactions, polls, pins, files, read state, search, and explicit AI triggers. |
| **Forum** | Subforums, threads, replies, voting, tags, bookmarks, moderation, and source-derived publication state. |
| **Synthesis** | Curation runs, source selection, claims, retries, destinations, results, feedback, and recovery. |
| **AI execution** | Model activities, tool calls, route metadata, citations, artifacts, terminal state, and operational audit. |
| **Media** | Uploads, generated images and video, previews, processing state, and durable attachment to product content. |
| **Capabilities and synchronization** | Deployment feature negotiation, client compatibility, durable table synchronization, and realtime delivery contracts. |

The boundaries are application modules rather than separately deployed services. This keeps transactions and domain invariants close to the data while preserving explicit ownership of behavior. Internal APIs provide the seams required for later extraction if load, deployment, or organizational boundaries justify it.

## PostgreSQL authority

PostgreSQL is the authoritative store for community content, workflow state, AI activity, and artifacts. The application does not treat a provider response, queue acknowledgment, socket event, or client cache as proof that durable product state exists.

Representative durable records include:

- accounts, memberships, communities, and roles;
- chat rooms, messages, reply relationships, reactions, polls, and files;
- forum containers, threads, replies, votes, bookmarks, and tags;
- synthesis runs, selected source messages, participants, destinations, results, and feedback;
- AI activities, model calls, tool calls, citations, warnings, and terminal state;
- artifacts and media jobs with pending, completed, failed, and delivery state;
- capability and synchronization metadata used by clients.

Ecto changesets and transactions enforce field, relationship, and lifecycle constraints before state is published to clients or background workers.

## Transactional workflow admission

Durable background work is created with the application state that identifies it. A synthesis request or AI turn should not produce a database record without scheduled work, nor should a queued job exist without the durable identity and scope required to execute it.

The application coordinates creation and enqueue operations so that retries converge on the same work. Oban uniqueness, explicit run or activity identifiers, and terminal-state checks prevent a retried request from publishing duplicate messages or artifacts.

A typical admission transaction establishes:

1. the triggering actor and product object;
2. the room, community, or forum scope;
3. the durable run or activity identity;
4. the initial lifecycle state;
5. the background job arguments required to continue;
6. any uniqueness key needed to collapse duplicate delivery.

The worker later claims or loads that state rather than rebuilding authority from job arguments alone.

## Authorization across domains

Authorization is evaluated through application functions that understand product relationships. It is not delegated to client routes, model prompts, or direct table access.

### Actor and community access

Account and membership state determines whether an actor may read a room, post a message, enter a forum, invoke an AI capability, or perform a moderation action. REST, LiveView, Channels, search, reactions, polls, and AI tools call the same domain-level checks.

### Source-derived restrictions

A forum thread synthesized from chat carries its source relationship. Effective read access considers both the destination forum and the originating room. The broader destination does not override the more restrictive source audience.

### Tool effects

AI tools are admitted with an actor and product scope. A model-proposed action is not executed because its schema parsed successfully. The action layer checks that the requested effect is available for the current surface and authorized for the relevant community object.

### Revalidation

Long-running work rechecks current state before reading or committing sensitive effects. A user leaving a room, a role change, a deleted source object, or a disabled feature can change what a previously admitted job may still do.

## Four classes of state

Flip uses different mechanisms for different consistency requirements.

| State class | Examples | Delivery and recovery model |
|---|---|---|
| **Durable product state** | Messages, threads, memberships, synthesis runs, AI activities, citations, artifacts. | PostgreSQL transactions and Ecto queries; reconstructable after reconnect or restart. |
| **Durable asynchronous state** | Queued AI turns, synthesis work, media generation, retries, continuation jobs. | Oban jobs tied to durable application identities and terminal-state checks. |
| **Ephemeral realtime state** | Presence, typing, immediate coordination notifications. | Phoenix Channels and PubSub; allowed to expire rather than replay. |
| **Client-local state** | Optimistic mutations, pending UI state, cached synchronized rows. | Reconciled against server transactions and Electric change delivery. |

Treating these classes separately avoids two common errors: persisting every transient signal as business data, or relying on transient sockets for state that must survive a reconnect.

## Realtime delivery

### Phoenix LiveView and Channels

LiveView serves the web product and uses server-side domain functions for commands and reads. Channels provide room and user topics for immediate events such as new content, presence, typing, and targeted notifications.

The socket layer publishes the result of committed application operations. It is not the source of truth for message, membership, workflow, or artifact state.

### Electric synchronization

React-based desktop and mobile clients subscribe to durable table shapes through Electric. Local optimistic mutations carry identities that can be reconciled with committed transactions and subsequent change delivery.

Electric handles durable state convergence. Channels remain appropriate for ephemeral signals and coordination that should not be represented as synchronized rows.

### Capability negotiation

Clients receive deployment capability metadata before exposing optional flows. Authentication methods, push notifications, deep links, uploads, reactions, synchronization, and AI features may differ by deployment or protocol version. The client should hide or disable unsupported behavior instead of discovering missing capability after the user acts.

## Failure containment

The modular application keeps failure domains visible even when components share one deployment.

- A model-provider failure marks or retries an AI activity; it does not change ordinary chat or forum authority.
- A failed media job retains its artifact identity and terminal error; it does not require the chat message itself to disappear.
- A disconnected client reconstructs durable state from PostgreSQL and synchronization rather than replaying every socket event.
- A crashed background worker can be retried because its application identity, scope, and prior state are durable.
- A stale or duplicate job exits when it finds an existing terminal result rather than publishing again.
- A missing optional provider or connector is represented through capability and health state rather than crashing unrelated product domains.

## Selected private implementation paths

| Path | Responsibility |
|---|---|
| `lib/flip/accounts.ex` and `lib/flip/accounts/` | Account, session, profile, and membership state. |
| `lib/flip/chat.ex` and `lib/flip/chat/` | Real-time community domain, rooms, messages, reactions, polls, and related lifecycle. |
| `lib/flip/forum.ex` and `lib/flip/forum/` | Forum containers, threads, replies, voting, and origin metadata. |
| `lib/flip/synthesis.ex` and `lib/flip/synthesis/` | AI turns, curation workflows, tools, artifacts, retries, and recovery. |
| `lib/flip/authz.ex` | Actor-aware access checks across product domains. |
| `lib/flip/llm/` | Durable model activity, tool-call, source-ledger, and audit records. |
| `lib/flip/capabilities.ex` | Deployment and client feature capability projection. |
| `lib/flip/sync.ex` | Durable client synchronization contracts and mutation reconciliation. |
| `lib/flip/application.ex` | OTP supervision and configured runtime services. |

[← Flip case study](../README.md)
