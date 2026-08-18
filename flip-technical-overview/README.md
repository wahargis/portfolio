# Flip — Technical Overview

**A real-time community product that connects live conversation, durable forum knowledge, and governed AI participation without losing authorship, permissions, or provenance.**

Flip is built around a simple product observation: chat is where communities think together, while forums are where useful knowledge remains findable. Existing products usually force a choice between the two or leave the transition to manual copy-and-paste. Flip treats them as one lifecycle and adds AI in roles that preserve, rather than blur, that distinction.

<img src="diagrams/system-context.svg" alt="Flip system context diagram" width="900" />

## The product model

A discussion begins in chat because immediacy, social context, and low-friction participation matter. When part of that discussion becomes worth preserving, a curation workflow can organize it into forum structure while retaining links to the messages and participants that produced it. The forum artifact becomes searchable and extendable without pretending it was written independently of the live exchange.

AI participates in two different ways:

| Role | Product contract |
|---|---|
| **Conversation curator** | Changes structure around human conversation while preserving source words and authorship. |
| **AI participant** | Creates new, visibly AI-authored replies, research, actions, and artifacts under product permissions. |

That separation is the central integrity decision. Curation cannot silently rewrite members; direct AI participation cannot hide behind the members’ voices.

## Why this is more than a chat-plus-forum interface

The hard part is the state and authority shared across the product:

- the same identity and membership rules govern chat, forum, internal retrieval, curation, and AI actions;
- a durable forum artifact can retain exact source-message and participant relationships;
- an AI answer can retrieve current external evidence or authorized internal discussion and persist citations as product objects;
- a chart, image, video, poll, or document result can have pending, completed, failed, and continued lifecycle rather than existing only as model prose;
- web, desktop, and mobile clients can remain responsive while converging on one server-authoritative state.

These are product contracts, not a list of model features.

## Architecture in one view

<img src="diagrams/service-container-map.svg" alt="Flip service and container architecture diagram" width="900" />

Flip is a modular Phoenix application with PostgreSQL as canonical state and Oban as the durable asynchronous work layer. Phoenix channels carry ephemeral interaction; Electric synchronizes durable client projections. The AI runtime is inside the product boundary: it receives product-derived identity and context, exposes server-authorized capabilities, routes hosted or local inference, isolates tool execution, and commits results through ordinary domain services.

The modular monolith is deliberate. Chat, forum, curation, citations, artifacts, authorization, jobs, and client synchronization benefit from shared transactions and direct relational provenance. Service extraction remains a scaling option, not a prerequisite for architectural seriousness.

## Two representative flows

### From a live discussion to durable knowledge

A room discusses a decision across several interleaved replies. The curator selects an eligible source set, proposes a coherent topic and forum destination, and submits a structured plan. Code verifies the source identities, authorship, destination, and duplicate state before applying the forum effect. Linkback and later participant correction retain their own durable lifecycle.

The result is not merely a summary. It is a forum object whose relationship to the original discussion remains inspectable.

### From a question to an evidence-aware AI reply

A member invokes an AI participant. Flip derives the actor/community scope, selects bounded context, and admits only the tools appropriate to that surface. The model can search, read sources, query authorized product content, or create a durable artifact. Citations and effects receive stable identities. A controlled terminal step produces the user-facing reply, which is validated and persisted under the AI identity before clients are notified.

The model supplies judgment and composition. The product supplies permission, evidence identity, lifecycle, and durability.

## Key invariants

1. Human-authored material remains attributed to humans.
2. AI-authored material remains explicitly attributed to the AI participant.
3. Protected internal retrieval follows the invoking actor’s visibility and fails closed without trusted scope.
4. Models choose among server-admitted capabilities; they do not create their own authority.
5. Citations, artifacts, and product actions have durable identities outside model prose.
6. Duplicate or retried background work converges on the intended effect.
7. Provider failure can degrade an AI capability without corrupting ordinary chat or forum state.
8. Clients can be optimistic, but the server remains authoritative for durable mutations and permissions.

## Product references

- Product: <https://flip.engineering>
- Synthetic technical environment: <https://flip.tech-demo.dev>
- Canonical source: <https://github.com/wahargis/flip>

<figure>
  <img src="assets/flip.engineering.png" alt="Flip product screenshot" width="760" />
  <figcaption>Flip product surface: live conversation and durable community structure.</figcaption>
</figure>

<figure>
  <img src="assets/flip.tech-demo.dev.png" alt="Flip technical environment screenshot" width="760" />
  <figcaption>Separate synthetic environment used to exercise public architecture scenarios.</figcaption>
</figure>

## Documentation path

| Question | Page |
|---|---|
| What is the system and why is it architecturally non-trivial? | [00 — Executive Overview](docs/00-executive-overview.md) |
| What product problem and user journeys shape the design? | [01 — Product and Problem](docs/01-product-and-problem.md) |
| Why a modular monolith, one database authority, jobs, and several realtime paths? | [02 — System Architecture](docs/02-system-architecture.md) |
| How does an AI event become an authorized, durable product outcome? | [03 — AI Participant Runtime](docs/03-agent-runtime.md) |
| How do retrieval, evidence, actions, and artifacts fit one capability contract? | [04 — Retrieval, Evidence, and Capability Plane](docs/04-retrieval-search-and-tools.md) |
| How are curation authorship and AI authorship kept distinct? | [05 — Curation, Authorship, and Provenance](docs/05-synthesis-and-provenance.md) |
| How do web/native clients remain immediate and converge on server truth? | [06 — Data, Realtime, and Client Convergence](docs/06-data-realtime-and-clients.md) |
| How can hosted/local models change without changing product semantics? | [07 — Model Routing as Execution Policy](docs/07-model-routing-and-inference.md) |
| How is public technical review separated from product authority? | [08 — Product and Synthetic Environment Boundary](docs/08-production-and-demo-topology.md) |
| How are deterministic contracts and model behavior evaluated differently? | [09 — Quality and Evaluation Strategy](docs/09-evaluation-testing-and-operations.md) |
| Which foundational decisions shape the system? | [10 — Architecture Decisions](docs/10-architecture-decisions.md) |
| What is implemented, under pressure, or deliberately limited? | [11 — Status, Pressure Points, and Limitations](docs/11-roadmap-and-known-limitations.md) |

Rendered diagrams are indexed in [diagrams/README.md](diagrams/README.md), stable decisions in [adr/README.md](adr/README.md), and synthetic scenarios in [demo/README.md](demo/README.md).

## Portfolio boundary

This case study selects the product semantics, architecture, lifecycles, decisions, and limitations needed for technical review. It does not duplicate the entire source tree or publish production data, credentials, prompt/persona configuration, abuse thresholds, or host-specific secrets.