# Flip

> A real-time community platform in which people and AI participate in conversation, and useful discussion can become durable, reviewable forum knowledge.

## At a glance

| | |
|---|---|
| **Product** | Community chat, forums, media, and scoped AI participation in one Phoenix application. |
| **Users** | Community members, moderators, and operators who need both immediate conversation and durable shared knowledge. |
| **Core problem** | Fast-moving chat is socially useful but difficult to retrieve, govern, or preserve. Adding AI increases the value of context while also increasing the consequences of context leakage, unclear authorship, and untraceable tool use. |
| **Engineering focus** | Real-time state, transactional background workflows, cross-context authorization, bounded AI context, provenance, and web/native-client compatibility. |
| **Primary implementation** | Elixir, Phoenix, LiveView and Channels, PostgreSQL/Ecto, Oban, and Electric-based client synchronization. |
| **Source** | [`wahargis/flip`](https://github.com/wahargis/flip) |

## The product problem

Most community software separates chat and forums into different products or treats one as a secondary view of the other. That leaves a structural gap:

- Chat is immediate, participatory, and suited to live coordination.
- Forums are durable, searchable, attributable, and better suited to accumulated knowledge.
- AI can help participants reason, retrieve, summarize, and create, but a useful AI response depends on context that may be private, temporary, or specific to one room.

Flip treats those concerns as one product architecture. Conversation remains conversation. Durable knowledge is created through an explicit synthesis lifecycle rather than by silently turning every message into an index. AI participates inside the same social and authorization model as the rest of the product instead of operating as an omniscient process outside it.

The resulting system is not “chat with an LLM added.” Its central work is preserving the meaning and boundaries of content as that content moves through live interaction, AI generation, asynchronous processing, review, and publication.

## Representative workflow

A typical path through the system looks like this:

1. **Members converse in a room.** Messages, replies, reactions, polls, media, membership, and room visibility are handled as real-time product state.
2. **A participant explicitly invokes Flip.** An `@flip` mention or reply can trigger an AI response where the feature is enabled. The trigger is visible in the conversation rather than inferred from ambient surveillance.
3. **Context is assembled within the room boundary.** The reply path revalidates the actor’s access, loads a bounded window of recent messages, follows only same-room reply ancestry, and can apply a room-specific briefing. It does not perform implicit cross-room recall.
4. **The AI responds as an identifiable participant.** Generation and tool activity are recorded. A reader-safe source ledger can expose the status and tool-use record associated with the posted reply without exposing the full administrative trace.
5. **Useful discussion can enter synthesis.** Selected chat content is claimed by an asynchronous workflow, processed into a structured artifact, and advanced through explicit run states.
6. **A durable forum artifact is created.** The resulting thread remains linked to its source context. A synthesis-origin thread cannot become more public than the channel from which it was produced.
7. **The community continues from durable state.** The thread can support later reading and discussion without requiring a future participant to reconstruct the original live room from an unbounded transcript.

This workflow is the product’s core value: it connects conversational participation, AI assistance, and community memory without collapsing them into one undifferentiated feed.

## Architecture

```text
Members and AI participants
          |
          v
Phoenix LiveView / Channels / API
          |
          v
Chat domain --------------------> Actor-aware authorization
  |                                      |
  | selected content                     | audience and origin checks
  v                                      v
Synthesis workflow ---- Oban ----> Forum domain
  |                                      |
  | AI generation and tools              | durable thread/reply state
  v                                      v
LLM activity + source ledger       Search / sync / client surfaces
```

### 1. Real-time community state

The chat domain owns rooms, messages, replies, memberships, reactions, polls, and the other state required for live participation. Phoenix Channels and LiveView provide immediate interaction, while the database remains the durable source of truth.

This matters because the AI path is not a separate demo endpoint. It receives and produces the same kinds of room-scoped content that human participants do, and it must obey the same visibility and lifecycle rules.

### 2. Synthesis as a durable workflow

Synthesis is modeled as a stateful background process rather than a single prompt call. Work is claimed, executed, retried, completed, and persisted through explicit run state. Transaction boundaries prevent the database artifact and the queued work from drifting apart.

That design addresses several practical failures:

- A request can be retried without silently creating duplicate durable content.
- An operator can distinguish pending, active, completed, and failed work.
- A worker crash does not erase the fact that work was claimed.
- Publication can preserve source references and the audience inherited from the originating room.

### 3. AI participation with bounded context

Flip’s AI response path is intentionally scoped. The system can use recent room messages and same-room reply ancestry to answer in conversational continuity, but it does not equate “the platform stores this” with “the model may read this.”

The context boundary is enforced before retrieval. Room access is rechecked for the triggering actor, platform machinery is excluded from the conversational window, and durable community memory is handled through the forum/synthesis model rather than hidden global recall.

### 4. Cross-context authorization

Chat and forum content have different domain models, but a forum artifact synthesized from chat still carries obligations from its origin. The authorization layer sits above those contexts and resolves the effective audience.

The most restrictive boundary wins. In particular, a thread derived from a private or restricted channel cannot become readable merely because the forum containing it has a broader default audience. That rule converts provenance into an enforceable security property rather than a descriptive field.

### 5. Product and client capability negotiation

Flip supports web and native-client surfaces whose available features depend on deployment configuration and protocol compatibility. Capability metadata lets clients conditionally expose authentication, push, deep-link, upload, reaction, emoji, and synchronization behavior instead of assuming every deployment has the same credentials or service configuration.

This is an example of a broader design principle in the project: optional capability should be represented explicitly, not discovered by letting a user encounter a broken action.

## Key design decisions

### AI is a participant, not an invisible observer

A visible mention or reply supplies a clear interaction boundary. It tells the user why the AI is speaking and limits the tendency to treat all community activity as ambient model input.

The design still supports richer product behavior—tool calls, room briefings, game-specific interaction, reactions, and synthesis—but those behaviors remain attached to explicit product state and policy.

### Live context and durable memory are different resources

Recent chat context is bounded and ephemeral. Durable knowledge is represented through forum and synthesis artifacts with source lineage. This avoids building a product whose “memory” is merely an ever-growing prompt assembled from whatever text happens to be available.

### Authorization is resolved at ingress and across domain boundaries

Read and act permission is centralized so REST, LiveView, Channels, search, polls, forum views, reactions, and AI tools do not each invent their own interpretation of access. Cross-context reads resolve the parent server, subforum policy, membership, role, and source-channel restriction.

### Background work has explicit ownership and terminal state

Synthesis work is claimed and completed transactionally, with stale-work recovery and retry behavior represented in code. This is more reliable than treating a queued job as proof that a durable artifact exists or treating a model response as proof that publication succeeded.

### AI output carries inspectable execution evidence

Administrative traces and reader-facing source ledgers serve different audiences. The full trace supports diagnosis; the public projection gives a user useful information about generation and tool outcomes without leaking internal arguments, provider metadata, or sensitive errors.

## Reliability and failure handling

The implementation treats common AI-product failure modes as expected operating conditions:

- **Duplicate submission:** creation and enqueue operations are coordinated transactionally, and work has explicit identities and statuses.
- **Worker interruption:** claimed runs can be identified and recovered rather than disappearing inside a transient process.
- **Authorization drift:** access is resolved when content is read or acted upon, not assumed from an old link or previously loaded page.
- **Context leakage:** reply context is same-room and actor-authorized, with bounded ancestry and a bounded recent-message window.
- **Tool or provider failure:** activity and tool-call outcomes are recorded and can be projected as warnings instead of being presented as an unqualified successful answer.
- **Partially configured deployments:** capability checks allow clients and endpoints to fail deliberately rather than expose unusable controls.

## Implementation evidence

| Source | What it demonstrates |
|---|---|
| [`lib/flip/chat.ex`](https://github.com/wahargis/flip/blob/main/lib/flip/chat.ex) | The central real-time community domain and the state AI participation must integrate with. |
| [`lib/flip/synthesis.ex`](https://github.com/wahargis/flip/blob/main/lib/flip/synthesis.ex) | Transactional synthesis creation, background-job coupling, run claiming, completion, retry, and recovery. |
| [`lib/flip/authz.ex`](https://github.com/wahargis/flip/blob/main/lib/flip/authz.ex) | Actor-aware authorization across chat, forum, search, polls, reactions, and AI-related reads; source-channel restrictions for synthesized content. |
| [`lib/flip/synthesis/ai_reply_detector.ex`](https://github.com/wahargis/flip/blob/main/lib/flip/synthesis/ai_reply_detector.ex) | Explicit AI triggers, authorized room-scoped context, bounded reply ancestry, recent-message grounding, and room briefings. |
| [`lib/flip/llm/source_ledger.ex`](https://github.com/wahargis/flip/blob/main/lib/flip/llm/source_ledger.ex) | Reader-safe projection of AI activity and tool-call evidence, separate from the detailed administrative trace. |
| [`lib/flip/capabilities.ex`](https://github.com/wahargis/flip/blob/main/lib/flip/capabilities.ex) | Deployment capability and client-compatibility metadata for optional authentication, push, deep-link, upload, reaction, and sync features. |

## Detailed technical overview

The case-study page is the orientation layer. The existing technical series provides the in-depth path without forcing a first-time reader to infer an order from filenames.

| Page | Focus |
|---|---|
| **[00 — Executive overview](docs/00-executive-overview.md)** | The product, its three defining design problems, and the shortest technical review path. |
| **[01 — Product and problem](docs/01-product-and-problem.md)** | User journeys, product invariants, and why AI participates inside the social system rather than above it. |
| **[02 — System architecture](docs/02-system-architecture.md)** | Modular-monolith boundaries, PostgreSQL authority, asynchronous work, client projection, scaling seams, and failure containment. |
| **[03 — AI participant runtime](docs/03-agent-runtime.md)** | Admission, bounded context, capability selection, tool execution, terminal composition, persistence, continuation, and failure semantics. |
| **[04 — Retrieval, evidence, and capability plane](docs/04-retrieval-search-and-tools.md)** | Internal/external retrieval, typed tools, trusted scope, durable citations, effects, retries, and security consequences. |
| **[05 — Curation, authorship, and provenance](docs/05-synthesis-and-provenance.md)** | The distinction between restructuring human conversation and creating new AI-authored content. |
| **[06 — Data, realtime, and client convergence](docs/06-data-realtime-and-clients.md)** | Durable, asynchronous, ephemeral, and local state across Phoenix web and React/native clients. |
| **[07 — Model routing as execution policy](docs/07-model-routing-and-inference.md)** | Product-owned inference contracts, surface-specific routing, adapters, fallback, local/hosted capacity, and evaluation. |
| **[08 — Product and synthetic environment boundary](docs/08-production-and-demo-topology.md)** | Public technical review without production data, credentials, or administrative authority. |
| **[09 — Quality and evaluation strategy](docs/09-evaluation-testing-and-operations.md)** | Deterministic invariants, model-route evaluation, end-to-end convergence, operations, and evidence discipline. |
| **[10 — Architecture decisions](docs/10-architecture-decisions.md)** | Major decisions, rationale, tradeoffs, rejected alternatives, and conditions for revisiting them. |
| **[11 — Status, pressure points, and limitations](docs/11-roadmap-and-known-limitations.md)** | Implemented scope, current architectural pressure, known limitations, and outcome-oriented next work. |

## What the project demonstrates

For a software or IT reviewer, Flip shows the design of a large stateful Phoenix application whose AI behavior is integrated with authorization, persistence, asynchronous work, and client contracts.

For an AI technical reviewer, the important point is not the presence of a model call. It is the surrounding control system: why generation is triggered, which context is eligible, how tool activity is recorded, how generated content becomes durable, and how source access continues to constrain the result.

For a product reviewer, Flip demonstrates a coherent answer to a recognizable user problem: communities generate valuable knowledge in conversation, but they need a deliberate path for preserving it without sacrificing the social immediacy of chat.

## Scope and boundaries

- Flip does not describe AI as a replacement for community governance or moderation.
- A synthesis run is a curation and publication workflow, not a claim that every conversation should be automatically summarized.
- The AI reply context is deliberately narrower than all data available to the application.
- The portfolio does not claim performance or community-scale metrics that are not established by repository evidence.
- This page focuses on the load-bearing product architecture. The implementation repository contains substantially more community, media, account, billing, game, notification, search, and client-support behavior than can be usefully inventoried here.

## Review paths

**Five minutes:** read **The product problem**, **Representative workflow**, and **Key design decisions**.

**Twenty minutes:** continue through **Architecture**, **Reliability and failure handling**, and the first five source links.

**Deep review:** inspect the linked implementation modules alongside the repository’s tests and domain-specific specifications.

[← Back to portfolio](../README.md) · [View source repository](https://github.com/wahargis/flip)
