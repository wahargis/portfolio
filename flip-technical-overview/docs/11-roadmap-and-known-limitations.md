# 11 — Status and Limitations

## Status model

This page separates implemented architecture from continuing work. It avoids issue inventories and implementation diaries because those age quickly and obscure the product’s actual contracts.

## Implemented capability domains

### Product core

- accounts, authentication, memberships, and authorization;
- real-time chat with replies, reactions, pins, presence/typing, read state, uploads, GIF/emoji support, and search;
- threaded forums with communities/subforums, replies, votes, bookmarks, tags, search, and origin metadata;
- notifications, settings, audit, and support/billing-related product surfaces;
- PostgreSQL-backed domain state and Oban-backed asynchronous workflows.

### Conversation curation

- scheduled/requested source selection;
- topic planning and forum placement;
- source-message and participant provenance;
- thread/reply creation, merge/deduplication, and linkback;
- feedback, recuration, and maintenance lifecycle;
- constrained curation-specific tool surface.

### AI participation

- mention/reply and product-event triggering;
- explicit AI participant/persona state;
- bounded iterative tool runtime;
- context-specific tool catalogs;
- actor/community-scoped internal retrieval;
- external research, source comparison, documents, structured data, charts, and rich artifacts;
- image/video/document/media workflows where configured;
- citations and source ledgers;
- terminal composition, invalid-output repair, and durable recovery/disclosure;
- asynchronous artifact continuation.

### Clients and deployment

- Phoenix web application and realtime paths;
- React/TypeScript client architecture;
- Electric-backed durable synchronization;
- native desktop packaging;
- mobile packaging target from the shared client;
- product and synthetic technical deployment roles;
- provider-compatible remote or local inference.

## Active architectural fronts

### Runtime decomposition

The AI reply runtime has accumulated many effect and recovery modes. The public architecture already separates trigger, context, routing, dispatch, evidence, terminal composition, and persistence; the implementation should continue moving toward correspondingly testable modules and typed envelopes.

### Capability registry coherence

As tool families grow, catalog membership, authorization, effect classification, timeout policy, and UI support must remain single-sourced. A tool that exists but is unreachable—or is exposed without the correct result renderer—is a contract defect.

### Client parity and offline semantics

Desktop and mobile packaging do not automatically provide identical behavior. Continuing work includes explicit offline command queues, conflict handling, push lifecycle, native integration tests, large artifact behavior, and clear parity statements.

### Evaluation

Deterministic tests are strong for authorization and lifecycle, but model-route quality requires versioned corpora, route matrices, citation semantic review, curation review, and drift detection tied to model/configuration changes.

### Media lifecycle

Long-running media workflows require robust cancellation, provider receipts, cost/attempt accounting, continuation deduplication, partial-chain recovery, and durable user control.

### Moderation and abuse

AI participation expands moderation requirements: prompt injection from retrieved content, abusive media requests, coordinated spam, impersonation, citation laundering, and manipulation of product actions. Product policy and enforcement must evolve alongside capability.

## Known limitations

### Modular-monolith complexity

The shared transaction and authorization boundary is valuable, but it can encourage large contexts and workers. Boundary tests and refactoring discipline are necessary to prevent a “single app” from becoming “no architecture.”

### External dependencies

Search, model, market/data, media, and document providers can be unavailable, change APIs, or produce inconsistent results. Flip normalizes failure but cannot remove upstream uncertainty.

### Citation scope

Quote verification can prove that a source contains a passage. It does not prove the passage is true, current, or sufficient for the model’s broader claim. Source comparison and semantic review remain necessary.

### Retrieval quality

Authorization-correct retrieval can still miss relevant content or rank poor context. Full-text, relational, and semantic strategies each have failure modes.

### Local inference constraints

Self-hosting improves control but introduces finite GPU capacity, context/latency tradeoffs, model capability differences, and operator burden. Product latency must respect the actual deployment.

### Offline behavior

Durable client synchronization supports reconnect and caching, but not every mutation has a defined offline conflict policy. The portfolio does not claim universal offline-first writes.

### AI behavior

Bounds, tools, and validation reduce failure impact; they do not make model reasoning deterministic. The system must retain honest uncertainty and user correction paths.

### Portfolio scope

The canonical source is public at <https://github.com/wahargis/flip>. This portfolio does not duplicate every implementation module, test, migration, provider adapter, or issue history. Source-level reproduction and implementation review belong in the source repository; this case study provides the stable architecture and product path through it.

## Next milestones by outcome

| Outcome | Architectural work |
|---|---|
| **More trustworthy research replies** | Route-specific evaluation, source-quality metadata, stronger claim-to-citation checks, disagreement rendering. |
| **More reliable action/artifact workflows** | Unified effect envelopes, cancellation, cost/attempt receipts, continuation state machine. |
| **Clearer product control** | Consolidated capability registry, administrator visibility, per-surface policy and audit. |
| **Stronger client resilience** | Explicit offline matrix, transaction reconciliation tests, native lifecycle and push integration. |
| **Lower runtime complexity** | Decompose the AI worker around stable trigger/context/tool/evidence/terminal/persistence contracts. |
| **Better community knowledge** | Curation review tools, duplicate detection, participant correction UX, source-retention policy. |
| **Safer scaling** | Queue partitioning, read-model tuning, workload isolation, database capacity evidence before service decomposition. |

## What would change the architecture

Potential future changes should be evidence-driven:

- split a worker role when queue/resource isolation is demonstrated;
- introduce a specialized search/vector service when PostgreSQL-based retrieval is measurably insufficient;
- add client-authoritative offline commands only after conflict semantics are specified;
- adopt a new model/provider only after surface-specific tool and terminal evaluation;
- expand autonomous actions only after authorization, audit, idempotency, and rollback are explicit.

The roadmap is therefore organized around product outcomes and contracts, not a sequence of fashionable model features.
