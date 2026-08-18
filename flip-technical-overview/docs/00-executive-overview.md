# 00 — Executive Overview

## System in one paragraph

Flip is a community product in which live chat, durable forum discussion, conversation curation, and explicit AI participants share one identity, permission, provenance, and client state model. Humans can talk at chat speed; valuable discussion can become searchable forum knowledge without erasing its source; and AI can research, answer, create artifacts, or perform permitted actions as an attributed product actor.

## The architectural problem

A conventional assistant integration assumes:

```text
prompt -> model -> text
```

Inside Flip, the model is participating in a multi-user stateful system. A useful answer depends on who invoked it, which community and room the request belongs to, what the actor may read, which capabilities are admitted, what external evidence supports the answer, and what durable effect the product should commit.

The same is true of curation. “Summarize this chat” is not enough when the durable result must preserve participant authorship, source navigation, correction history, and forum identity.

Flip therefore treats model use as one stage inside product-owned workflows rather than an alternate backend.

## Three design problems define the system

### Preserve the path from conversation to knowledge

Chat and forums remain different interaction models, but they share data and provenance. The curation path can reorganize eligible discussion into a forum artifact while retaining the source messages and authors that produced it. Readers get durable structure without a fabricated standalone authorship story.

### Make AI a governed participant

An AI participant has an explicit identity and bounded product role. Flip selects context, computes the capability catalog, enforces internal visibility, isolates tools, records citations and artifacts, and validates the terminal reply before persistence. The model interprets and composes; the product owns authority and lifecycle.

### Keep asynchronous and client state coherent

Curation, research, generated media, and continuation can outlive an HTTP request. Oban gives those workflows durable identity and retry state. PostgreSQL remains canonical. Electric synchronizes recoverable client projections; channels carry ephemeral interaction. A provider or client disconnect cannot become an excuse to publish state that never committed.

## Architectural center of gravity

Flip is a modular Phoenix monolith with one PostgreSQL authority. This keeps cross-domain transactions, authorization, source relationships, jobs, search, and client synchronization coherent. It also creates internal complexity that must be managed through real domain APIs and decomposition of large orchestration modules.

The system can scale by Phoenix nodes, worker roles, storage, database tuning/replication, and workload isolation before the product semantics need to be split across services.

## The product’s two AI authorship contracts

| Workflow | What the model may do | What the product must preserve |
|---|---|---|
| **Conversation curation** | Identify topics, order material, choose a destination, and add bounded bridge context. | Human words, participant identities, eligible source set, source links, correction lineage. |
| **AI participation** | Compose a new answer, research, select admitted tools, and propose permitted effects. | Explicit AI identity, actor/community scope, evidence, artifact/action identity, terminal lifecycle. |

This distinction is more important than any individual model or tool.

## Why the architecture is reviewable

The implementation expresses product correctness in durable structures and deterministic boundaries:

- relational source and authorship identities;
- server-side authorization and catalog computation;
- isolated, typed tool/effect dispatch;
- durable citations and artifacts;
- explicit job uniqueness and terminal state;
- client reconciliation with canonical transactions;
- provider adapters behind a product-owned inference contract.

Model quality still requires evaluation and human correction. The architecture makes those failures visible and containable rather than claiming to remove them.

## Suggested review path

Read [System Architecture](02-system-architecture.md), [AI Participant Runtime](03-agent-runtime.md), and [Curation, Authorship, and Provenance](05-synthesis-and-provenance.md) for the core system. Use [Capability Plane](04-retrieval-search-and-tools.md), [Client Convergence](06-data-realtime-and-clients.md), and [Model Routing](07-model-routing-and-inference.md) for the deeper execution boundaries. [Status and Limitations](11-roadmap-and-known-limitations.md) states where the current implementation remains under pressure.