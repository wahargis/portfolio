# Flip technical overview

Flip is a real-time community application with integrated AI participants. The product includes accounts, membership, chat, forums, search, media, background work, and web and native clients. AI capabilities operate inside those product systems and use the same identity, visibility, persistence, and lifecycle rules.

The technical overview focuses on three execution paths:

1. a direct AI turn inside conversation or another product surface;
2. a durable multi-stage operation such as research, document work, image or video generation, or another asynchronous artifact;
3. curation of selected conversation into durable forum structure while retaining source authorship and access constraints.

## Direct AI execution

```text
product event
  -> actor, AI identity, origin, and feature state are resolved
  -> bounded product context and admitted retrieval are assembled
  -> model route and turn-specific tools are selected
  -> model and tool rounds produce evidence, actions, or artifacts
  -> terminal output and references are validated
  -> one product result is committed
  -> web and native clients receive durable and realtime updates
```

A provider never receives a general product database or an unrestricted action interface. Internal retrieval uses trusted actor and community scope. Effectful tools call application services that validate targets, assign durable identities, and record lifecycle state. The final reply is stored separately from the model's working protocol and provider stream.

## Durable asynchronous execution

Image, video, document, and other provider-backed operations may not finish inside one model turn. Flip creates a durable job or artifact before the provider work completes. The object records its inputs, route, attempt, progress, result, and failure state.

When the operation reaches a terminal state, the application can start one deduplicated continuation under the original product scope. The continuation loads the completed artifact and resumes the product workflow without depending on the original worker or provider connection.

```text
agent requests long-running capability
  -> pending artifact and job are committed
  -> provider or worker updates durable state
  -> completion or failure event is recorded
  -> one continuation is admitted under original scope
  -> reply or later workflow uses the stored result
```

## Conversation curation

Flip treats curation of human conversation differently from direct AI authorship.

A direct AI reply creates new content attributed to an AI identity. A curation run selects source messages, proposes topics and destinations, validates the plan, creates or updates forum objects, preserves participant and message references, and records linkback and feedback state. Generated bridge text can be used where the product allows it, but it is not substituted for the source authorship of selected messages.

Source relationships also affect access. A forum item derived from restricted conversation cannot become readable only because the destination forum is broader.

## Main system areas

| Area | Responsibility |
|---|---|
| **Identity and authorization** | Human and AI identities, community membership, roles, room and forum visibility, origin-aware reads, and product action checks. |
| **Chat and forum domains** | Durable messages, replies, reactions, polls, threads, posts, source relationships, and user-visible lifecycle state. |
| **Agent runtime** | Admission, context, model routing, tool rounds, terminal composition, activity records, continuation, and failure handling. |
| **Retrieval and evidence** | Internal product search under actor scope, web discovery, direct source reading, documents, PDFs, structured data, citations, and evidence records. |
| **Capabilities and effects** | Typed read and write tools, product actions, charts, files, generated media, and other application services. |
| **Background execution** | Oban jobs, durable runs, retries, stale-work recovery, provider polling or callbacks, and continuation after terminal artifacts. |
| **Clients and realtime delivery** | LiveView, Channels, APIs, durable synchronization, optimistic reconciliation, desktop and mobile packaging, and capability negotiation. |
| **Operations and evaluation** | Structured events, health and failure state, deterministic product tests, route evaluation, security checks, and synthetic technical scenarios. |

## State ownership

PostgreSQL owns durable product, agent, artifact, and workflow state. Oban and supervised application processes execute work against those records. Phoenix Channels and PubSub carry ephemeral state such as presence, typing, and transient progress. Clients own local interaction state such as drafts and open views until a server command is accepted.

This separation supports restart and reconciliation:

- a disconnected client can reload from durable state;
- a failed background process does not erase its run or artifact;
- an asynchronous provider can complete after the originating request;
- a model-route failure does not become a chat or forum transaction;
- an optimistic client object converges on a canonical server identity.

## Engineering boundaries

The system maintains the following boundaries:

- Product identity and permissions are resolved by the application, not inferred by the model.
- Internal retrieval is scoped before data is selected.
- Provider adapters do not define product tools or persistence.
- Model tool calls are requests to typed application capabilities.
- Durable effects require committed product records.
- Direct AI authorship and curation of human content have different provenance.
- Realtime delivery is not the source of truth for durable product state.
- Long-running work can resume from stored artifact and checkpoint state.
- Public technical material excludes product data, credentials, private host configuration, provider keys, and administrative access.

## Documentation path

- [Product and agent use cases](01-product-and-problem.md)
- [System architecture](02-system-architecture.md)
- [Agent runtime](03-agent-runtime.md)
- [Retrieval, evidence, and tools](04-retrieval-search-and-tools.md)
- [Curation and provenance](05-synthesis-and-provenance.md)
- [Data, realtime, and clients](06-data-realtime-and-clients.md)
- [Model routing and inference](07-model-routing-and-inference.md)
- [Product and synthetic environment boundary](08-production-and-demo-topology.md)
- [Testing, evaluation, and operations](09-evaluation-testing-and-operations.md)
- [Architecture decisions](10-architecture-decisions.md)
- [Status and limitations](11-roadmap-and-known-limitations.md)

[Back to the Flip case study](../README.md)
