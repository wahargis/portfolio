# 02 — System Architecture

## Architectural thesis

Flip is a modular Phoenix monolith because its differentiating behavior depends on relationships that cross ordinary product boundaries: a chat message can become source material for a forum artifact; an AI reply can cite external evidence and attach a durable artifact; a permission change must affect search, synchronization, and tool use consistently.

Those relationships are easier to make correct when identity, authorization, transactions, background work, and provenance share one application and one canonical database. The architecture is modular to preserve ownership and reasoning boundaries, but it is not distributed merely to look sophisticated.

<img src="../diagrams/service-container-map.svg" alt="Flip service and container map" width="900" />

## The system is organized around four boundaries

### 1. Product authority

The Phoenix application owns the social system: accounts, membership, chat, forum, moderation, settings, notifications, and the durable representation of AI-authored content. Domain contexts expose public operations rather than allowing clients, models, or background workers to mutate tables arbitrarily.

The important boundary is not the number of contexts. It is that every effect passes through the context that understands its invariants. Sending a message, creating a forum thread, attaching a citation, and changing a room policy are different domain commands even when they share a database transaction.

### 2. Durable asynchronous work

Oban carries work that cannot or should not complete inside an HTTP request: conversation curation, AI turns, linkback, recuration, provider-backed artifacts, continuation, and maintenance.

A job is durable coordination state, not merely a retry wrapper. It records causal identity, attempt state, and the terminal product condition the workflow owes. A worker returning successfully is insufficient if the intended reply, linkback, or artifact state was never committed.

### 3. AI capability plane

The AI runtime sits inside the product rather than beside it. It receives product-derived context and authorization, exposes a server-computed capability set, routes model calls, isolates tool execution, persists citations and artifacts, and commits the final result through ordinary domain services.

This prevents the model provider from becoming a second product backend. Models reason and choose among admitted actions; Flip decides what can be read, what can be changed, and what becomes durable.

### 4. Client projection

Web, desktop, and mobile clients project one product state through three mechanisms with different semantics:

- HTTP/domain commands for authoritative mutations;
- Electric synchronization for recoverable durable records;
- Phoenix channels/PubSub for ephemeral presence, typing, and transient progress.

The split is intentional. A committed message must be recoverable after disconnect; a typing indicator should not become permanent database history.

## Why one PostgreSQL authority matters

Consider a chat discussion that becomes durable forum knowledge. The system may need to create a curation run, create or update a forum thread, preserve source-message and participant relationships, attach structural context, enqueue linkback, and later accept participant correction.

In Flip, the durable part of that transition can be expressed with relational identities and transactions:

```text
chat messages + authorship
        |
        v
validated curation plan
        |
        v
forum artifact + source relationships
        |
        +--> linkback job
        +--> feedback / recuration lineage
```

Foreign keys and uniqueness constraints make provenance part of the data model rather than an optional JSON annotation. PostgreSQL also supplies full-text search, Oban state, and the logical-replication foundation used by client synchronization.

This choice concentrates responsibility in the database and migration discipline. That is preferable to introducing distributed consistency, duplicated authorization, and cross-service provenance before scale requires them.

## A representative cross-domain flow

An externally researched AI reply illustrates how the boundaries cooperate:

1. A chat message is committed under the invoking member and room.
2. A trigger creates one deduplicated AI job with causal identifiers.
3. The runtime derives actor/community scope and selects bounded conversation context.
4. The capability plane admits external retrieval and drafting tools appropriate to that surface.
5. Tool execution retrieves sources and persists citation identities.
6. The model composes an attributed reply using those identities.
7. Output validation rejects protocol leakage or invalid evidence references.
8. Chat persists the reply and its citation/artifact relationships.
9. Durable synchronization and ephemeral notifications update clients.

No layer independently owns the whole flow. Correctness comes from explicit contracts between them.

## Domain boundaries are defined by invariants

The most consequential boundaries are:

| Boundary | Invariant it protects |
|---|---|
| **Accounts / authorization** | The server, not the client or model, decides identity and visibility. |
| **Conversation** | Chat chronology, replies, authorship, and room membership remain coherent. |
| **Durable knowledge** | Forum structure can outlive chat velocity while retaining origin and correction history. |
| **AI effects** | AI-authored content, citations, artifacts, and actions remain attributed and authorized. |
| **Client projection** | Optimistic and synchronized views converge on canonical server state. |

The private implementation may contain large modules as the product has grown. That is an implementation pressure to decompose around these contracts, not evidence that the domains should become network services.

## Failure containment

The architecture is designed so capability failure does not automatically become product failure:

- a model endpoint can be unavailable while ordinary chat and forum writes continue;
- one tool can fail honestly while the AI turn uses already gathered evidence;
- a linkback failure does not erase a committed forum artifact;
- a media provider can leave a durable failed artifact rather than a broken message;
- a sync disconnect does not change PostgreSQL authority;
- a Phoenix process can restart under BEAM supervision without inventing durable success.

Database unavailability is different because the database is the product authority. In that case Flip must reject or queue writes rather than publish phantom state.

## Scaling seams

The modular monolith does not require one process or one machine forever. Phoenix nodes can scale horizontally; Oban roles can be isolated; read models and media storage can be separated; PostgreSQL can be tuned, replicated, or partitioned.

A service boundary is justified when a domain demonstrates an independent scaling, release, security, or team-ownership requirement. It is not justified merely because a module has a name.

## Security boundary

Several rules follow from the architecture:

1. Clients cannot choose their authorization scope.
2. Model tool calls are parsed as data and dispatched through known handlers.
3. Internal retrieval requires trusted actor/community scope and fails closed without it.
4. External content is untrusted input and passes URL, network, size, and rendering controls.
5. Provider credentials remain server-side.
6. Administrative configuration cannot be rewritten through AI-authored content.
7. Durable publication occurs only after the relevant domain transaction succeeds.

## Deliberate tradeoff

A modular monolith requires active boundary discipline. The alternative would make the same product semantics depend on network availability, duplicated policy, and eventual consistency. Flip accepts internal refactoring pressure in exchange for stronger product coherence and provenance.