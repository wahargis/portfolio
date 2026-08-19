# Flip — Technical Overview

**A real-time community product that connects live conversation, durable forum knowledge, and governed AI participation without losing authorship, permissions, or provenance.**

Flip begins with a product problem rather than an assistant interface. Communities do most of their thinking in chat because chat is immediate, contextual, and socially easy. The useful result is often difficult to recover later because explanations, decisions, and disagreements remain buried in chronology. Forums preserve knowledge, but asking participants to stop a live exchange and manually reconstruct it into a polished thread creates too much friction.

Flip treats chat and forum as two stages of one community-knowledge lifecycle. Valuable discussion can become durable, searchable forum structure while retaining the messages, reply relationships, and people that produced it. AI participates inside that same product boundary: it can help curate discussion, answer questions, research current information, create artifacts, or perform permitted product actions, but it does so under explicit identity, authorization, evidence, and lifecycle rules.

<img src="diagrams/system-context.svg" alt="Flip system context diagram" width="900" />

## At a glance

| Concern | Flip’s answer |
|---|---|
| **Who it is for** | Communities that want the speed of chat, the durability of forums, and useful AI participation without collapsing all three into an undifferentiated feed. |
| **What a member can do** | Talk in live rooms, continue durable forum discussions, retrieve prior community knowledge, ask an AI participant to research or act, inspect citations and artifacts, and navigate from curated material back to its source. |
| **What the system owns** | Identity, membership, authorization, chat/forum state, curation provenance, AI participant identity, tool admission, citations, artifacts, background jobs, notifications, and client convergence. |
| **Primary implementation shape** | A modular Phoenix application with PostgreSQL authority, Oban jobs, Phoenix realtime delivery, Electric-backed durable client synchronization, and web/native clients. |
| **Source boundary** | The product/server implementation remains private. This public case study is derived from the implementation, tests, schemas, and product behavior without linking to the private repository. |

## What using Flip looks like

A member can ask a question in a live room while a larger discussion is unfolding. Flip records the message under that member and room, derives the community context and visibility available to the request, and invokes an explicitly configured AI participant. The AI can read authorized community material, search current external sources, compare evidence, or create a product-native artifact. The final answer appears as an AI-authored message with durable citation and artifact relationships rather than as an opaque provider transcript.

Later, the room reaches a decision worth preserving. A curation workflow selects the eligible source messages, proposes a topic and forum destination, and submits a structured plan. Code validates the message identities, participant relationships, destination, and duplicate state before applying any durable effect. The resulting forum material remains connected to the original discussion, and later participant correction or recuration has its own causal history.

Those two flows use models differently. The first creates new AI-authored content. The second changes the structure around human-authored conversation. Flip makes that distinction visible in the data model and product behavior rather than relying on a prompt to preserve it.

## The product model

A discussion begins in chat because immediacy, social context, and low-friction participation matter. When part of the discussion becomes worth preserving, a curation workflow can organize it into forum structure while retaining links to the messages and participants that produced it. The forum artifact becomes searchable and extendable without pretending it was written independently of the live exchange.

AI participates in two primary roles:

| Role | What the model contributes | What the product must preserve |
|---|---|---|
| **Conversation curator** | Topic recognition, ordering, destination judgment, and bounded structural context. | Human wording and authorship, the eligible source set, source navigation, destination identity, duplicate protection, feedback, and recuration lineage. |
| **AI participant** | Original answers, research synthesis, tool selection, artifact requests, and permitted product actions. | Explicit AI identity, invoking actor/community scope, admitted capabilities, evidence identities, effect receipts, terminal state, and durable publication. |

That separation is the central integrity decision. Curation cannot silently rewrite members; direct AI participation cannot hide behind the members’ voices.

## Why this is more than a chat-plus-forum interface

The difficult work is not rendering two types of screen. It is preserving shared authority and state across the product:

- the same identity and membership rules govern chat, forum, internal retrieval, curation, and AI actions;
- a durable forum object can retain exact source-message and participant relationships;
- an AI answer can retrieve current external evidence or authorized internal discussion and persist citations independently of model prose;
- a chart, image, video, poll, document result, or product action can have pending, completed, failed, and continued lifecycle;
- background retries converge on one intended effect rather than duplicating a reply, artifact, or linkback;
- web, desktop, and mobile clients can remain responsive while converging on one server-authoritative state;
- a provider or tool can fail without corrupting ordinary community state or fabricating completion.

These are product contracts, not a catalog of model features.

## Architecture in one view

<img src="diagrams/service-container-map.svg" alt="Flip service and container architecture diagram" width="900" />

Flip is a modular Phoenix application with PostgreSQL as canonical state and Oban as the durable asynchronous work layer. Phoenix channels carry ephemeral interaction; Electric synchronizes recoverable durable client projections. The AI runtime sits inside the product boundary: it receives product-derived identity and context, exposes server-authorized capabilities, routes hosted or local inference, isolates tool execution, and commits results through ordinary domain services.

The modular monolith is deliberate. Chat, forum, curation, citations, artifacts, authorization, jobs, and client synchronization benefit from shared identities, transactions, and direct relational provenance. The application can still scale by Phoenix node, worker role, model/provider capacity, storage, and database tuning before those semantics need to be split across network services.

### Product authority

Phoenix contexts own the commands that change community state. A client, model, or worker does not receive generic authority to mutate tables. Sending a message, creating a forum thread, attaching a citation, publishing a poll, changing membership, and completing an artifact are separate domain effects because they protect different invariants.

### Durable asynchronous work

AI turns, curation, linkback, recuration, media generation, continuation, and maintenance can outlive one HTTP request or model call. Their jobs carry stable causal identities, retry state, and terminal obligations. A worker returning successfully is not enough if the intended reply, forum update, or artifact state never committed.

### Governed AI capability plane

The server computes the capabilities available to each AI turn after it knows the surface, configured providers, invoking actor, community, and object context. Protected internal search receives trusted actor/community scope from the runtime and fails closed if that scope is missing. Tool exceptions and timeouts are converted into honest model-visible failures rather than silently terminating the reply worker or widening authority.

### Client projection

Web and native clients use different transport semantics for different kinds of state: authoritative commands for mutations, durable synchronization for recoverable records, channels for ephemeral presence and progress, and local state for drafts or optimistic placeholders. The client can be immediate without becoming a second product authority.

## A product-owned AI lifecycle

An AI reply is treated as a product transaction rather than `prompt -> model -> text`:

1. A product event identifies the invoking actor, room or forum, parent object, and AI participant.
2. A deduplicated durable job establishes one causal execution identity.
3. The runtime selects bounded, authorized context rather than dumping all available history.
4. The server computes the capability catalog appropriate to that surface.
5. The model reasons and may request isolated retrieval, analysis, artifact, or product-action tools.
6. Evidence and effects receive durable identities and receipts.
7. A controlled terminal composition step produces the actual user-facing candidate rather than exposing internal tool protocol.
8. Validation checks authorship, evidence/artifact references, rendering safety, and terminal structure.
9. The product commits the reply and relationships under the AI identity.
10. Durable synchronization and ephemeral notifications update clients only after the relevant state exists.

If asynchronous media or another provider-backed effect finishes later, Flip starts one deduplicated continuation using the durable artifact state. It does not keep an imaginary model process alive while waiting.

## Two representative flows

### From a live discussion to durable knowledge

A room discusses a decision across several interleaved replies. The curator selects an eligible source set, proposes a coherent topic and forum destination, and submits a structured plan. Code verifies the source identities, authorship, destination, and duplicate state before applying the forum effect. Linkback and later participant correction retain their own durable lifecycle.

The result is not merely a summary. It is a forum object whose relationship to the original discussion remains inspectable.

### From a question to an evidence-aware AI reply

A member invokes an AI participant. Flip derives the actor/community scope, selects bounded context, and admits only the tools appropriate to that surface. The model can search, read sources, query authorized product content, analyze structured information, or create a durable artifact. Citations and effects receive stable identities. A controlled terminal step produces the user-facing reply, which is validated and persisted under the AI identity before clients are notified.

The model supplies interpretation and composition. The product supplies permission, evidence identity, lifecycle, and durability.

## What is implemented rather than merely proposed

The implementation includes the ordinary community product, the conversation-to-forum path, AI participant workers, context-gated tools, durable run and operation records, citation and artifact state, background continuation, capability negotiation for clients, and separate product/synthetic deployment authority.

Several code-level choices make the architectural claims concrete:

- AI and curation services are supervised as product processes and workers rather than invoked from an isolated demo script.
- Curation runs record status, model/prompt identity, expected and committed operations, plan fingerprints, latency/tokens, and terminal state.
- Curation operations retain their source-message set, action, target thread, counts, and uniqueness within the run.
- AI reply workers preserve triggering identity, use bounded tool rounds and deadlines, recover from provider/tool failure, and commit through the conversation domain.
- Tool availability changes by surface, and protected searches receive trusted scope through the dispatch path rather than model-supplied identity.
- Optional client capabilities are advertised by the server so a desktop or mobile client does not assume a feature merely because UI code exists.

The portfolio does not present every tool, provider, or worker as an equal product feature. Those implementation details matter where they establish the authority, lifecycle, and failure contracts described above.

## Key invariants

1. Human-authored material remains attributed to humans.
2. AI-authored material remains explicitly attributed to the AI participant.
3. Protected internal retrieval follows the invoking actor’s visibility and fails closed without trusted scope.
4. Models choose among server-admitted capabilities; they do not create their own authority.
5. Citations, artifacts, and product actions have durable identities outside model prose.
6. Duplicate or retried background work converges on the intended effect.
7. Provider failure can degrade an AI capability without corrupting ordinary chat or forum state.
8. Clients can be optimistic, but the server remains authoritative for durable mutations and permissions.
9. Curation changes structure around human content; it does not convert human conversation into unattributed AI prose.
10. A worker or provider response is not user-visible success until the corresponding product state commits.

## Product references and source boundary

- Product: <https://flip.engineering>
- Synthetic technical environment: <https://flip.tech-demo.dev>
- Source: private by design; no private repository is linked from this portfolio.

<figure>
  <img src="assets/flip.engineering.png" alt="Flip product screenshot" width="760" />
  <figcaption>Flip product surface: live conversation and durable community structure.</figcaption>
</figure>

<figure>
  <img src="assets/flip.tech-demo.dev.png" alt="Flip technical environment screenshot" width="760" />
  <figcaption>Separate synthetic environment used to exercise public architecture scenarios without production data or authority.</figcaption>
</figure>

The public documentation is grounded in the private implementation but deliberately omits credentials, production data, private messages, prompt/persona configuration, abuse thresholds, deployment secrets, and source paths that are not needed to evaluate the architecture.

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

This case study selects the product semantics, architecture, lifecycles, decisions, implementation evidence, and limitations needed for technical review. It is not a substitute source tree and does not expose production authority.