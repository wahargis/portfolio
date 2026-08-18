# 11 — Status, Pressure Points, and Limitations

## What is implemented

Flip is already a broad product system rather than an architecture proposal. The public description groups implemented behavior by the product contract it serves instead of reproducing every module or tool name.

| Area | Implemented system |
|---|---|
| **Community product** | Authenticated communities with real-time chat, threaded forums, membership/authorization, search, notifications, settings, moderation-related controls, and durable PostgreSQL/Oban state. |
| **Conversation-to-knowledge path** | Source selection, model-assisted topic/destination planning, validated forum creation/append/merge, source-message and participant provenance, linkback, feedback, and bounded recuration. |
| **AI participation** | Explicit AI identities, product-event triggering, context selection, route policy, server-computed capabilities, internal/external/document/data retrieval, citations, durable artifacts/actions, terminal composition, recovery, and asynchronous continuation. |
| **Client and deployment** | Phoenix web, React/TypeScript client architecture, Electric durable synchronization, native desktop packaging, mobile packaging path, provider-compatible local/hosted inference, and a separate synthetic technical environment. |

This breadth is real, but it creates architectural pressure. The next work is primarily about making the existing product easier to reason about, evaluate, and control—not attaching more disconnected model features.

## Current pressure points

### AI runtime decomposition

The direct-reply runtime coordinates triggers, context, providers, many tool/effect types, evidence, terminal composition, artifacts, games/specialized lanes, and recovery. The public architecture already separates those contracts conceptually; the implementation should continue moving toward smaller typed components around admission, working state, effects, terminal state, and persistence.

The goal is not microservices. It is making failures local and tests meaningful inside the modular monolith.

### Capability coherence

As the capability plane expands, several facts must remain single-sourced: whether a tool is reachable on a surface, what trusted scope it requires, whether it is read-only or effectful, how long it may run, what durable identity it returns, and which client renderer understands the result.

A tool that exists but is unreachable, or is exposed without the right authorization or artifact UI, is not harmless inventory—it is a broken contract.

### Route and model evaluation

Authorization and lifecycle can be tested deterministically. Model behavior cannot. Route changes need versioned surface-specific evidence for tool correctness, citation use, curation validity, terminal composition, failure honesty, latency, and local capacity behavior.

Without that discipline, provider flexibility becomes silent product drift.

### Client parity and offline behavior

A shared React codebase and native wrappers do not guarantee behavioral parity. Desktop and mobile have different background, notification, storage, permission, and connectivity constraints.

Offline mutation remains deliberately command-specific. Further work requires explicit idempotency, conflict, ordering, revocation, and user-visible pending semantics rather than a blanket “offline-first” claim.

### Long-running artifact lifecycle

Media and other provider-backed work need reliable cancellation, provider/request receipts, attempt and cost accounting, partial-chain state, deduplicated continuation, and durable user control. The product already represents asynchronous artifacts; the pressure is to make every provider path conform to one comprehensible lifecycle.

### Moderation and abuse

AI participation expands the product threat model: retrieved prompt injection, cross-community data probing, abusive media generation, spam, impersonation, citation laundering, and misuse of product actions. Capability growth must remain coupled to authorization, audit, rendering, and policy rather than treated as a model-only problem.

## Known limitations

### The modular monolith can concentrate complexity

Shared transactions and policy are valuable, but large contexts and workers can become difficult to change safely. The architecture depends on maintaining real domain APIs and decomposing internal orchestration where complexity accumulates.

### Upstream providers remain uncertain

Model, search, document, structured-data, and media providers can fail, change contracts, or return inconsistent results. Flip can normalize failure and preserve product state; it cannot eliminate upstream uncertainty.

### Citation verification is not truth verification

The system can verify that a selected passage exists in a retrieved source and is attached to a claim. It cannot automatically establish that the source is correct, current, or sufficient for the model’s inference.

### Retrieval can be authorized and still be poor

Full-text, relational, semantic, external, and document retrieval each have recall and ranking failures. Permission correctness prevents leakage; it does not guarantee that the best context was selected.

### Local inference trades external dependence for operational burden

Self-hosting provides control and data locality but introduces finite slots, model capability differences, GPU/process failure, context/latency tradeoffs, and operator responsibility. The product must respect actual deployment capacity.

### Offline semantics are incomplete by design

Durable synchronization supports caching and reconnect, but not every command has a defined offline conflict policy. Flip does not claim that all writes can originate authoritatively on a device.

### Model behavior remains non-deterministic

Bounds, tools, evidence, terminal validation, and correction paths reduce the impact of failures. They do not turn model reasoning into deterministic software. Honest uncertainty and participant correction remain required product features.

### The portfolio is curated, not a source mirror

The canonical implementation is public at <https://github.com/wahargis/flip>. The portfolio groups stable product contracts, architecture, and limitations for review rather than duplicating every implementation module, test, migration, provider adapter, or issue history. Source-level verification and reproduction belong in the canonical repository.

## Next work by outcome

| Desired outcome | Architectural direction |
|---|---|
| **More trustworthy research replies** | Stronger route-specific evaluation, claim-to-citation review, source-quality/disagreement representation, and correction feedback. |
| **More reliable actions and artifacts** | One effect envelope and lifecycle across product actions and provider-backed work, with cancellation and receipt semantics. |
| **Clearer administrator control** | Single capability/policy representation with surface visibility, audit, and explicit degradation state. |
| **More resilient clients** | Command-specific offline queues, convergence tests, permission revocation, push/native lifecycle, and large-artifact behavior. |
| **Lower AI-runtime complexity** | Decompose around stable admission, context, dispatch, evidence, terminal, and persistence contracts. |
| **Better durable community knowledge** | Stronger curation review, duplicate detection, participant correction, and source-retention behavior. |
| **Evidence-based scaling** | Isolate queue or storage workloads only after capacity and failure evidence justifies the boundary. |

## What would materially change the architecture

The current shape should be revisited if measured load requires independent data/worker planes, if a domain needs regulatory or release isolation, if offline conflict semantics justify client-authoritative commands, if PostgreSQL-based retrieval is measurably inadequate, or if provider-neutral standards remove meaningful adapter work.

The roadmap is organized around product outcomes and contracts. It should not become a list of fashionable model capabilities or implementation issue numbers.
