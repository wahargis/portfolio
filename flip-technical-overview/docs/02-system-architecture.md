# System architecture

Flip is a Phoenix application with product contexts for identity, communities, chat, forums, search, media, AI activity, curation, and other application services. PostgreSQL stores durable state. Oban and supervised processes execute background work. LiveView, Channels, APIs, and durable client synchronization deliver state to web, desktop, and mobile surfaces.

The architecture is organized around product transactions and long-running workflows rather than around model providers.

## System context

![Flip system context](../diagrams/system-context.svg)

The application communicates with several external systems:

- local and hosted model endpoints;
- web search and direct source services;
- document, image, video, storage, and media providers;
- push, email, authentication, and connector services where configured;
- desktop and mobile clients using the server's API, realtime, and synchronization contracts.

External services do not own product identity, permissions, object lifecycle, or publication. They receive bounded requests and return results to application-owned records.

## Application structure

![Flip service and container map](../diagrams/service-container-map.svg)

### Product contexts

Accounts, communities, chat, forums, search, media, and related contexts own their schemas and state transitions. AI operations call these contexts through application services rather than writing product tables directly.

This keeps product rules in the same code paths for human and AI actions. For example, creating a poll or forum item from an AI turn uses the same target validation and persistence model as another server command, with additional activity and provenance state.

### AI execution services

The agent runtime handles turn admission, context assembly, route selection, capability admission, model and tool rounds, terminal validation, activity records, and continuation. Retrieval, providers, tool execution, and artifact services are separate modules behind this runtime.

The runtime does not own every product feature it can invoke. It owns the execution contract and calls the product context that owns each effect.

### Background workflows

Oban jobs and supervised workers execute work that should not depend on an HTTP request remaining open. This includes curation, provider polling or callbacks, generated media, document processing, notifications, repair, and continuation after terminal artifacts.

A background job refers to durable product or run state. The queue entry is not treated as the only record of the operation.

### Data and persistence

PostgreSQL stores:

- accounts, memberships, rooms, messages, forums, posts, reactions, polls, and media;
- AI identities, activities, tool calls, source and citation records, route and usage state;
- curation requests, runs, source relationships, feedback, and publication state;
- generated and uploaded artifacts, provider attempts, progress, terminal results, and errors;
- job-facing identities, idempotency keys, retries, and continuation state;
- client synchronization records and other durable application state.

Transactional boundaries are placed around product effects that must commit together. A job can be inserted in the same transaction as the durable request it will execute. A forum result and its source relationships can be committed together. Later stages such as linkback can have separate state so they can be retried without duplicating the forum mutation.

## Main execution paths

### Direct AI reply

```text
Phoenix command or message event
  -> resolve actor, AI identity, community, origin, and feature state
  -> assemble bounded context
  -> admit route and tools
  -> execute model and tool rounds
  -> validate terminal reply and references
  -> commit AI activity and message
  -> publish durable and realtime updates
```

### Asynchronous artifact

```text
agent or product command
  -> commit pending artifact and job
  -> worker or provider updates attempt and progress
  -> commit completed or failed artifact
  -> schedule one continuation when applicable
  -> continuation loads stored result and resumes product flow
```

### Conversation curation

```text
selected source messages
  -> commit curation request and run
  -> load authorized source state
  -> produce and validate destination plan
  -> create or update forum objects and source relationships
  -> record linkback, feedback, and repair state
```

### Native-client command

```text
client command with local transaction identity
  -> server authenticates and authorizes
  -> product context commits canonical object
  -> command response and durable sync may arrive in either order
  -> client reconciles by canonical and transaction identity
  -> realtime events update ephemeral state
```

## Authorization placement

Authorization is resolved at product ingress and enforced again at protected reads and effects.

The application uses trusted server values for actor, community, origin object, and AI identity. Model-generated tool arguments can specify a user request such as a search term or poll options, but they cannot replace the trusted scope.

Cross-context reads consider both the destination and the origin. A curation-derived forum object can retain a restriction from its source room. Search and retrieval use the same visibility logic as direct product reads rather than indexing protected text into a globally available agent store.

## Provider boundary

The product constructs a provider-independent request containing selected context, instructions, tools, output requirements, deadlines, and route metadata. Adapters translate the request to each provider and normalize:

- streaming text and structured events;
- tool calls and tool-result continuation;
- finish, stop, refusal, timeout, and protocol errors;
- usage and route metadata where available;
- asynchronous provider identifiers and callbacks for media work.

Provider adapters do not decide which product data is eligible or which effect should be committed.

## Client architecture

The server-rendered web application can use the same Phoenix contexts directly. React-based clients use authenticated APIs, realtime channels, and durable synchronization.

The server remains authoritative for:

- authentication and active membership;
- product object identity and lifecycle;
- visibility and action permission;
- accepted command state;
- durable job, activity, and artifact state;
- capability metadata for the current deployment and client protocol.

Clients can own drafts, open views, and optimistic placeholders. These objects remain provisional until the server accepts the command and supplies the canonical identity.

## Scaling and isolation

The current architecture scales by separating workload classes before separating all product domains:

- web and API requests can scale across application nodes;
- Channels and PubSub distribute realtime events;
- Oban queues separate interactive follow-up, curation, media, document, and maintenance work;
- per-community or per-origin uniqueness prevents duplicate workflows;
- provider and route concurrency can be limited independently;
- large media objects use external object storage while PostgreSQL retains metadata and lifecycle;
- local inference can be replaced or supplemented by hosted routes without changing product persistence.

A domain becomes a candidate for independent deployment only when its contract and operating needs are stable enough to justify distributed failure, authorization, and transaction boundaries.

## Failure containment

- A model or provider failure can fail one AI activity without invalidating ordinary chat or forum state.
- A worker crash leaves the durable request, job, run, or artifact available for retry or repair.
- A realtime disconnect does not remove the committed object; clients reload from durable state.
- A failed linkback does not require deleting a completed forum result.
- An unavailable optional connector or provider is removed from capability admission rather than causing unrelated product actions to fail.
- A malformed model result is rejected before becoming a product effect.
- A revoked membership is applied at read and action time even when a client has an old local copy.

[Previous: Product and agent use cases](01-product-and-problem.md) · [Next: Agent runtime](03-agent-runtime.md)
