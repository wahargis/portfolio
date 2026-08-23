# Product and agent use cases

Flip is a community product in which people communicate through realtime rooms and durable forums. AI identities can participate in those product surfaces, use retrieval and tools, create artifacts, and assist with curation. The application must keep user-visible behavior, data access, and durable effects consistent across human actions, model turns, background work, and web and native clients.

## Product roles

| Role | Relevant product state |
|---|---|
| **Community member** | Account, community membership, room and forum access, messages, replies, reactions, polls, uploads, AI requests, artifacts, and notification state. |
| **Moderator or operator** | Product configuration, feature availability, community and content controls, AI identity configuration, activity and failure inspection, and repair of incomplete workflows. |
| **AI identity** | Attributed product participant with selected instructions, route policy, permitted product surfaces, tools, and persistent activity records. |
| **Background worker** | Executes a specific durable job or run under stored application scope and updates its lifecycle state. |
| **Web or native client** | Renders authorized product state, submits commands, receives durable synchronization and ephemeral events, and reconciles optimistic UI state. |

An AI identity is not a privileged system account. It is a configured product actor whose behavior is admitted by the server for a specific turn. The invoking human actor and origin object still determine which data and effects are allowed.

## Use case: AI participation in conversation

A member mentions or replies to an AI identity in a room. The server verifies that the feature is available, that the member can read and participate in the room, and that the selected AI identity is enabled for the surface.

The runtime then assembles a bounded context. This can include recent room messages, same-room reply ancestry, direct references, room-specific information, and selected artifact state. Platform events and content outside the current room are not included merely because they are stored by the application.

The model can answer directly or use admitted tools. A final reply is attributed to the AI identity and stored as a normal product object with related AI activity and source state. Users see a product participant and its reply, not a raw provider transcript.

## Use case: internal and external research

A user can ask for research that requires product-native records, external sources, documents, or structured data.

Internal retrieval is performed under trusted actor and community scope. External research separates discovery from direct source reading so the final response can refer to the material actually retrieved. Document and PDF tools maintain a relationship between the selected file or passage, the tool result, and the final response. Structured data operations return typed tables, statistics, or charts rather than unverifiable prose.

The product stores relevant citations, tool outcomes, files, and artifacts outside the model context. A later user can inspect the visible evidence and result state without requiring the original provider conversation.

## Use case: product actions

AI participation can include product-native actions such as creating a poll, preparing a forum item, attaching a file, generating a chart, or invoking another admitted application service.

The model does not commit these effects directly. It produces a typed request. The server applies the trusted actor and origin scope, validates the request against domain rules, performs the operation through the owning application context, and returns the durable result to the turn.

This allows the AI to participate in product workflows without giving the provider arbitrary access to internal APIs or database records.

## Use case: generated media and long-running work

Image and video generation, editing, and related media operations often complete asynchronously. The user should see a durable pending artifact, progress or provider state, and an explicit completion or failure result.

The product creates the artifact before the provider call becomes an accepted effect. A worker or callback updates the artifact and job. A completed result can start one continuation that posts the result or resumes a larger workflow. A failure remains visible and can be retried or repaired without duplicating already completed stages.

## Use case: chat-to-forum curation

A member or moderator selects a portion of conversation for durable organization. A curation run identifies source messages, proposes topic or destination structure, validates the proposal, and creates or updates forum content.

The durable result retains the selected participants and source-message relationships. The application can use generated bridge text where appropriate, but it does not rewrite source authorship. Linkback and feedback connect the forum result to the conversation and support later correction or recuration.

The destination's default visibility does not override a more restrictive source boundary.

## Use case: web, desktop, and mobile clients

The Phoenix application and React-based clients share product state but do not use one transport for every kind of update.

Durable objects are stored in PostgreSQL and delivered through server-authoritative APIs and synchronization. Realtime channels carry presence, typing, transient progress, and other non-replayable state. Clients can use optimistic placeholders for responsive interaction, but the server response or durable synchronization supplies the canonical identity and final status.

Capability negotiation tells a client which authentication, upload, push, deep-link, synchronization, and AI features are available in the current deployment and protocol version. A client does not expose a control merely because another deployment supports it.

## Product requirements for agent execution

The use cases above impose specific system requirements.

### Identity and scope

Every turn must have a resolved invoking actor, AI identity, community, origin object, and feature configuration. Protected retrieval and effects must use these server-owned values.

### Bounded context

The runtime must select relevant product and external context under a working budget. It cannot assume that all available data belongs in the model prompt or that a large context window removes ranking and privacy requirements.

### Typed capabilities

Tools need schemas, trust classification, authorization, timeouts, lifecycle, idempotency where required, and stable result identities. Read-only, effectful, and asynchronous capabilities require different control paths.

### Durable evidence and artifacts

Citations, files, charts, generated media, polls, and other artifacts need product records independent of provider text. A final response can refer only to results the application can identify and render.

### Recoverable background work

Long-running operations need jobs or runs with explicit pending, active, completed, failed, and cancellation state. The application must be able to retry or continue after a worker or server restart.

### Realtime convergence

Web and native clients need a consistent path from optimistic state to canonical durable objects. Reordered command responses, synchronization, and realtime events must not create duplicates or preserve revoked access.

### Inspectable operation

Operators need to determine which route ran, which tools were called, which durable effects occurred, why an artifact failed, whether a continuation was scheduled, and which evidence is safe to show to a user.

## Public technical boundary

The public Flip material documents system behavior, architecture decisions, data and control boundaries, and synthetic scenarios. It does not publish private implementation source, product data, user content, credentials, provider keys, host configuration, prompt and persona state, or administrative endpoints.

[Previous: Executive overview](00-executive-overview.md) · [Next: System architecture](02-system-architecture.md)
