# Flip Synthetic Technical Environment

The synthetic environment exposes representative Flip product and runtime behavior without production accounts, communities, messages, credentials, moderation authority, or deployment state.

**Product:** <https://flip.engineering>  
**Synthetic technical environment:** <https://flip.tech-demo.dev>

<img src="../assets/flip.tech-demo.dev.png" alt="Flip synthetic technical environment" width="100%" />

## Environment boundary

The product and synthetic deployments can run the same application code and public client contracts, but they use separate authority and data roots:

| Boundary | Product environment | Synthetic environment |
|---|---|---|
| **Accounts and communities** | Real users, memberships, moderation roles, and community configuration. | Generated users, roles, rooms, forums, and social state. |
| **Content** | Product chat, forum, document, and media records. | Seeded or generated conversations, documents, sources, and artifacts. |
| **Credentials and providers** | Production provider, storage, notification, and operational credentials. | Independent restricted credentials or simulated providers appropriate to public demonstration. |
| **Administration** | Real operational and moderation authority. | Restricted synthetic controls with no path into production state. |
| **Observability** | Production telemetry and private diagnostics. | Sanitized lifecycle and execution state suitable for public technical inspection. |

No production record is required to demonstrate the architecture. Synthetic data carries the same relevant types and relationships while remaining disposable and non-sensitive.

## Product behavior represented

### Live community state

The environment includes chat rooms, replies, reactions, polls, forums, threaded discussion, search, and media state. These surfaces demonstrate the same distinction between durable product objects and ephemeral presence or typing signals used by the full product.

### Explicit AI participation

Synthetic conversations can include visible AI triggers and AI-authored responses. The runtime still resolves an actor, room, eligible context, admitted tools, provider route, activity record, citations, artifacts, and terminal state. The environment does not use a privileged global context merely because the data is synthetic.

### Retrieval and citations

Generated internal content, configured public sources, and synthetic documents provide evidence for retrieval flows. Citations and source-ledger records retain source identity, retrieval status, and access behavior rather than appearing only as formatted text in a response.

### Conversation curation

Seeded chat can move through a synthesis run into a forum thread with source-message, participant, room, and destination relationships. Curated human conversation remains distinguishable from newly authored AI content.

### Typed artifacts

The environment can represent pending, completed, and failed images, video, documents, charts, and other rich outputs. Artifact identity and lifecycle remain durable even when the backing provider or generated file is synthetic.

### Client convergence

Web and native-compatible state follows the same server authority, optimistic mutation, durable synchronization, and ephemeral Channel split described in the client architecture. Reconnection reconstructs durable state without replaying expired presence and typing events.

### Controlled failures

Synthetic provider, retrieval, tool, artifact, and workflow failures can be represented without affecting production. The resulting product state distinguishes the stage that failed, preserves completed durable work, avoids duplicate effects, and supplies an honest terminal status.

## Publicly inspectable state

Depending on the deployed synthetic scenario, the environment can expose:

- chat and forum objects;
- AI-authored messages and triggering relationships;
- synthesis-run and source relationships;
- reader-safe citations and source ledgers;
- typed artifact status and previews;
- visible pending, completed, failed, and recovered workflow states;
- sanitized tool and provider outcomes;
- realtime client updates.

Administrative traces remain restricted even in the synthetic environment. Public execution evidence is projected through reader-safe product records rather than unrestricted logs or raw provider payloads.

## Architecture references

- [Product and domain model](../docs/00-product-and-domain.md)
- [System and data architecture](../docs/01-system-and-data-architecture.md)
- [AI participant runtime](../docs/02-ai-participant-runtime.md)
- [Retrieval, tools, and artifacts](../docs/03-retrieval-tools-and-artifacts.md)
- [Clients and deployment](../docs/04-clients-and-deployment.md)
- [Testing, operations, and current status](../docs/05-testing-operations-and-status.md)

[← Flip case study](../README.md)
