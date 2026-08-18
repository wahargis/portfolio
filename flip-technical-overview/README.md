# Flip — Technical Overview

Flip is a real-time community product built around a simple product thesis: **conversation should be able to stay live without becoming disposable**.

Chat is where people actually work things out. Forums are where useful knowledge becomes discoverable and durable. Most products force a choice between those interaction models or bolt one onto the other. Flip treats them as two states of the same community knowledge system, with AI used to help bridge the gap without obscuring who said what or how a conclusion was reached.

<img src="diagrams/system-context.svg" alt="Flip system context diagram" width="900" />

## The product problem

The interesting part of Flip is not that it contains an AI assistant. It is that AI is inserted into an existing multi-user product where authorship, permissions, provenance, and durable state already matter.

That changes the design substantially.

A model cannot simply be handed the whole database because it wants more context. It cannot turn a user’s words into its own summary and still claim the same provenance. It cannot invoke a product action because a prompt told it that the action is allowed. And a generated answer that cites current information has to preserve enough evidence for another user to understand where that information came from.

Flip is built around those constraints.

## Two different uses of AI

The most important distinction in the system is between **curation** and **participation**.

Curation helps move useful discussion from chat into durable forum structure. The AI can identify topics, organize source material, and provide limited structural context, but the resulting artifact remains connected to the original conversation and preserves human authorship.

Participation is different. An AI identity can answer a question, conduct research, create an artifact, or take a permitted product action. In that case the output is explicitly AI-authored and enters the product as such.

Those two paths are deliberately separate because conflating them creates exactly the ambiguity the product is trying to avoid: did the system reorganize what people said, or did the AI say something new?

## AI as a governed product actor

A direct AI reply is handled as a product transaction, not as a raw model call.

The server determines who invoked the AI, which community the request belongs to, what internal content that actor may access, which external or product-native tools make sense in that context, and what effects are actually authorized. The model chooses within that admitted surface; it does not define the surface itself.

This is especially important for internal retrieval. A community assistant becomes a data-leak mechanism if “search the conversation” is implemented as a global semantic query. Flip carries the invoking actor and community scope into retrieval and fails closed when that scope is missing.

The same principle applies to actions. The AI may decide that creating a poll, generating an artifact, or performing another product action would help, but the effect still passes through server-side domain rules.

## Durable provenance rather than decorative citations

Flip treats provenance as product state.

For a synthesized forum artifact, provenance means preserving the relationship to source messages and participants. For an AI-authored answer, provenance may mean a quote-backed web or document citation. For a generated artifact, it means preserving the request and result relationship so that the artifact is not just an opaque attachment emitted by a provider.

Those records outlive the model context that created them. That is the useful distinction between evidence and prose that merely looks sourced.

## Architecture

<img src="diagrams/service-container-map.svg" alt="Flip service and container architecture diagram" width="900" />

Flip is implemented as a modular Phoenix application over PostgreSQL rather than as a collection of microservices. Chat, forum, synthesis, identity, authorization, background AI work, and provenance share transactional relationships that are more valuable than artificial service separation at the project’s current scale.

Oban owns durable asynchronous work such as synthesis and AI turns. Phoenix channels handle ephemeral interaction such as presence and typing. Electric is used where native clients need durable synchronized state. Those mechanisms are intentionally different because the state they carry has different semantics.

The client does not become an alternate source of truth simply because it can work optimistically. Durable writes remain server-authoritative and are reconciled back to the client.

## Why the AI runtime is bounded

Tool-using models are useful precisely because they can decide how to gather information or construct a response. The surrounding runtime exists to stop that flexibility from turning into undefined system behavior.

A turn has explicit lifecycle and resource bounds. Tool calls are checked against the server-computed capability set and executed in isolated tasks. Tool failures are returned as structured failures the model can reason about rather than crashing the whole reply path or encouraging the model to invent missing results. Long-running artifact workflows can persist state and continue after asynchronous completion instead of holding one model call open indefinitely.

The important architectural idea is not a particular round limit or provider. It is that the model’s autonomy is contained inside a product-owned execution envelope.

## What a reviewer should take away

Flip demonstrates a product architecture in which AI is neither a cosmetic chatbot nor an all-powerful backend. It participates in a social system with the same seriousness given to any other privileged actor: explicit identity, scoped access, constrained effects, durable provenance, failure handling, and clear authorship.

The chat/forum synthesis problem is what gives those choices practical meaning. The product is trying to preserve how communities actually reach knowledge, not merely generate more text about it.

## Product references

- <https://flip.engineering>
- <https://flip.tech-demo.dev> — separate synthetic technical environment

<figure>
  <img src="assets/flip.engineering.png" alt="Flip product screenshot" width="760" />
</figure>

## Deeper architecture

The detailed pages below cover the implementation where it is useful to understand a design decision rather than to enumerate features:

- [System architecture](docs/02-system-architecture.md)
- [Agent runtime](docs/03-agent-runtime.md)
- [Retrieval, search, and tools](docs/04-retrieval-search-and-tools.md)
- [Synthesis and provenance](docs/05-synthesis-and-provenance.md)
- [Data, realtime, and clients](docs/06-data-realtime-and-clients.md)
- [Model routing and inference](docs/07-model-routing-and-inference.md)
- [Architecture decisions](docs/10-architecture-decisions.md)
- [Status and limitations](docs/11-roadmap-and-known-limitations.md)

The implementation is private; this portfolio publishes the architecture and engineering reasoning rather than private source, credentials, prompt content, or internal operating detail.
