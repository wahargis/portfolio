# 02 — System Architecture

## Architectural style

Flip is a **modular Phoenix monolith** with a shared PostgreSQL database and explicit product contexts. The design favors strong cross-domain consistency, a single authorization model, and one operational release boundary over premature service decomposition.

<img src="../diagrams/service-container-map.svg" alt="Flip service and container map" width="900" />

## Component map

```text
Web / native clients
  |-- HTTP mutations and reads
  |-- Electric shape subscriptions
  `-- Phoenix channels for ephemeral realtime state
           |
           v
Phoenix application
  |-- Accounts / authorization / audit
  |-- Chat
  |-- Forum
  |-- Synthesis and AI participant runtime
  |-- Notifications
  |-- Media and artifacts
  |-- Settings / support / billing surfaces
  |-- API and synchronization controllers
  |-- Oban workers
  `-- telemetry and operational controls
           |
           +--> PostgreSQL / Ecto / search indexes
           +--> Oban durable jobs
           +--> Phoenix PubSub
           +--> embedded Electric synchronization
           +--> provider-compatible LLM endpoints
           `--> external search, data, document, and media services
```

## Domain contexts

| Context | Owns |
|---|---|
| **Accounts** | Users, authentication, sessions/tokens, profile state, memberships. |
| **Authorization** | Actor/community/room permission decisions and capability checks. |
| **Chat** | Rooms, messages, reply relationships, reactions, pins, memberships, read state, chat search. |
| **Forum** | Communities/subforums, threads, replies, votes, bookmarks, tags, forum search. |
| **Synthesis** | Curation runs, AI reply lifecycle, personas, tool catalogs, citations, source ledgers, recuration, continuation. |
| **Notifications** | Durable notification state and per-user delivery projections. |
| **Media / artifacts** | Uploads, generated artifacts, image/video/document outputs, lifecycle and attachment records. |
| **Audit / operations** | Material product actions, configuration changes, and runtime evidence. |
| **Settings / support** | Feature and room configuration, supporter/billing integration, product preferences. |

Context boundaries are application boundaries, not independent services. They communicate through public domain functions and durable relationships rather than each maintaining a private copy of shared state.

## Why one PostgreSQL database

### Cross-domain transactions

A synthesis operation may need to:

- create a run record;
- create a forum thread;
- create replies;
- preserve source-message relationships;
- attach citations/artifacts;
- update linkback state;
- enqueue follow-up work.

Keeping these records in one database permits transactional integrity and straightforward compensation when later asynchronous stages fail.

### Relational provenance

Source relationships are naturally relational:

```text
chat message
  -> synthesis selection
  -> forum thread/reply
  -> source ledger
  -> participant feedback
  -> recuration
```

Foreign keys and uniqueness constraints prevent provenance from becoming an optional JSON convention.

### Search and synchronization

PostgreSQL provides full-text indexing and the replication basis for Electric synchronization. The architecture can add specialized retrieval services where justified without moving canonical product state out of the relational store.

### Operational simplicity

One durable store supports backup, migration, audit, background jobs, and synchronization. This does not imply one node forever; it avoids introducing distributed consistency before product scale requires it.

## Asynchronous architecture

Oban owns durable asynchronous work. Representative categories include:

- room crawling and topic extraction;
- synthesis and linkback;
- recuration and maintenance;
- direct AI replies and reactions;
- document/media/artifact workflows;
- scheduled enrichment;
- continuation after provider completion;
- cleanup and stale-state repair.

A background job is not considered successful merely because a worker process returned. The workflow records the product effect and terminal state required by its contract.

### Idempotence and uniqueness

Jobs that can create duplicate user-visible content use uniqueness keys tied to the causal event. Long-running continuation workflows can key uniqueness on chain identity rather than the entire argument payload when several terminal events must converge on one continuation.

### Retry semantics

Retry policy depends on the effect:

- transient provider failure can retry with bounded backoff;
- a durable disclosure or completed reply should return success and not duplicate;
- invalid output may receive one bounded model-authored repair;
- a missing authorization scope should not be retried as if it were an outage;
- an irrecoverable artifact failure should persist failure, not remain indefinitely “running.”

## Realtime architecture

Flip uses different transports for different semantics.

| Mechanism | Best suited to |
|---|---|
| **HTTP mutation** | Authoritative commands with explicit validation and response. |
| **Electric shape stream** | Durable tables/read models that clients must synchronize and reconcile. |
| **Phoenix channel/PubSub** | Presence, typing, transient progress, and targeted low-latency events. |
| **Oban event completion** | Durable asynchronous changes later projected through sync or channels. |

Using Electric does not eliminate channels. Persisting every presence event would be wasteful; treating durable message state as an ephemeral channel event would make recovery and offline behavior fragile.

## AI subsystem boundaries

The AI subsystem contains several cooperating but separable concerns:

| Component | Responsibility |
|---|---|
| **Trigger detector/enqueuer** | Convert product events into one eligible background job. |
| **Context assembler** | Select conversation, room, forum, persona, artifact, and prior-turn state. |
| **Model client/router** | Normalize provider-compatible requests, capability differences, retries, and circuit behavior. |
| **Tool catalog** | Compute definitions admitted for this surface and deployment. |
| **Tool dispatcher** | Validate, authorize, execute, time-bound, isolate, and normalize tool results. |
| **Citation/artifact services** | Mint durable identities and provenance outside model prose. |
| **Terminal composer** | Obtain the user-facing draft under finish/repair rules. |
| **Output validator** | Reject blank output, protocol leakage, invalid artifact references, or unsafe shape. |
| **Persistence/publisher** | Commit attributed product content and notify clients. |
| **Telemetry/recovery** | Preserve terminal reason, latency/error class, and bounded recovery path. |

A single large worker may coordinate these concerns in the private implementation, but the architecture treats them as contracts to prevent one prompt loop from becoming the entire system model.

## Feature gating

Chat, forum, synthesis, media, support, and provider-backed capabilities can be enabled independently subject to dependency rules. Gating occurs at several layers:

- routes and UI;
- background queues and schedules;
- tool definitions;
- configuration validation;
- external-provider availability;
- actor/community authorization.

Disabling a feature should remove its active surface and background work, not merely hide a button.

## Deployment topology

The default topology can remain compact:

```text
reverse proxy / TLS
        |
Phoenix release
  |-- web/API/channels
  |-- Oban execution
  |-- embedded sync service
        |
PostgreSQL
        |
optional external or local model/data/media providers
```

Horizontal Phoenix nodes, separate worker roles, read scaling, or media object storage can be introduced at higher load without changing the core domain model.

## Security boundaries

1. Clients never choose their own authorization scope.
2. Tool calls are parsed as data and dispatched through known handlers.
3. Internal search requires an origin actor and community.
4. External fetchers apply URL/network/content controls.
5. Provider credentials remain server-side.
6. Generated HTML/markdown/media passes product rendering and sanitization rules.
7. Administrative configuration is separate from model-authored content.
8. Logs and source ledgers avoid leaking credentials or private cross-community data.

## Deliberate tradeoff

A modular monolith concentrates code and requires discipline in domain APIs. The alternative—distributed services—would add network failure, duplicated authorization, eventual consistency, and more difficult provenance. Flip chooses the former until observed scale or team boundaries justify the latter.
