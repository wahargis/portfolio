# 04 — Retrieval, Evidence, and the Capability Plane

## The design problem is not “give the model tools”

Inside a multi-user product, a tool is a capability contract. It must define who can invoke it, what data it can see, whether it is read-only or effectful, how failure is represented, and what durable evidence or receipt survives after the model turn.

Flip’s capability plane exists to preserve those semantics while still letting an AI participant research broadly, reason over structured data, create artifacts, and act inside the product.

<img src="../diagrams/retrieval-source-citation-flow.svg" alt="Retrieval, evidence, and citation flow" width="900" />

## Four capability classes

### Evidence acquisition

These tools obtain information the model does not already possess:

- external search followed by direct webpage or historical-source reading;
- actor-scoped chat and forum retrieval;
- document and PDF extraction;
- typed data providers for markets, economics, policy, conflict, environment, research, trade, and related domains.

The architectural distinction is between **discovery** and **evidence**. A search result identifies a candidate source; the reader retrieves the actual page or document passage. Internal retrieval returns product identities under the invoking actor’s visibility. Structured providers retain dates, units, missing values, and provenance rather than flattening everything into prose.

### Derived analysis

Comparison, transformation, charting, rich-data rendering, and research-session operations derive a result from identified inputs. Their output carries the relationship to those inputs so a chart or comparison remains inspectable rather than becoming an unexplained image or paragraph.

### Durable artifacts and product actions

Image/video generation and editing, polls, selected platform actions, and other effectful tools create state outside the model conversation. They run through product or provider services and return a durable request, artifact, action, or receipt identity.

This identity is what makes retries, pending state, cancellation, continuation, permissions, and rendering possible. A model-authored string saying “I created it” is not an effect.

### Terminal composition

The drafting capability is deliberately part of the control surface. It converts a working model state into the candidate user-facing reply. Separating it from ordinary tool calls prevents internal planning or provider protocol from leaking into the product.

## Catalog computation is authorization architecture

The runtime does not expose one universal registry. It computes the catalog from:

```text
surface and product role
  + feature / provider availability
  + actor and community authorization
  + object context
  + deployment policy
  = schemas admitted for this turn
```

A curation worker, personal participant, forum enricher, ordinary room participant, and specialized game turn have different jobs. Narrow surfaces can receive replacement catalogs so they do not inherit unrelated research or media capabilities.

This is why “the tool exists in code” does not mean “every model can call it.” Reachability, authorization, effect class, renderer support, and lifecycle must agree.

## Trusted scope follows execution

Internal retrieval demonstrates the critical pattern:

```text
reply runtime derives actor/community scope
        |
        v
scope is captured by the dispatch closure
        |
        v
child task calls a domain query
        |
        v
membership and visibility predicates are applied
```

The model’s arguments cannot choose a different actor or community. If the trusted scope is absent, the result is an explicit empty/denied envelope that tells the model not to fabricate a product record or citation.

## Dispatch isolates both failure and authority

A parsed call is matched to a known handler, validated, and executed in a bounded task. The wrapper normalizes exceptions, process exits, throws, timeouts, and malformed results.

<img src="../diagrams/governed-tool-execution.svg" alt="Governed tool execution" width="900" />

A failure result should give the model enough information to continue honestly without exposing internal stack traces or silently broadening access. The reply loop survives a broken tool; the product remains responsible for deciding whether any effect actually committed.

## Evidence becomes a durable citation

Flip’s citation flow separates source discovery from claim rendering:

1. A search, document, internal query, or typed provider returns an evidence envelope.
2. A bounded passage or record set is selected.
3. The server persists source metadata, the selected support, and a stable citation identity.
4. The model places that identity near the claim it supports.
5. Final validation confirms the identity exists and is visible in the current product context.
6. The UI renders a source ledger independently of provider-specific citation syntax.

Quote verification can establish that the represented passage occurs in the retrieved source. It cannot prove the source true or the model’s broader inference warranted. Source quality, date, disagreement, and claim scope remain interpretation problems and should remain visible.

## Retrieval is fit to the evidence type

Flip does not force every source through one vector-search abstraction:

- product content benefits from authorization-aware full-text and relational retrieval;
- external current information requires search plus direct reading;
- large documents need page/section extraction;
- structured data should remain typed;
- social context is often selected by reply, room, author, and provenance relationships;
- semantic/vector retrieval is useful where it measurably improves recall.

Vector similarity is one signal, not the product’s memory or permission model.

## Result budgeting preserves reopenability

Tool output can exceed a useful model context. The runtime can cap records, paginate, summarize with source identities, return next-request hints, or persist full data as an artifact while providing a compact model view.

The important rule is that compaction must not remove the identity needed to cite, reopen, or inspect the result. A shorter answer is useful; an untraceable answer is not.

## Effects and retries

Read operations, derived analysis, product actions, and provider-backed artifacts have different replay risks. Retrying a search is usually cheap. Retrying a poll publication, media charge, or message action may duplicate cost or state.

Effectful operations therefore use causal identities and idempotency appropriate to the domain. Asynchronous operations expose pending, completed, and failed state and can re-enter the AI runtime through one continuation rather than relying on an open model session.

## Security consequences

The capability design implies several hard boundaries:

- retrieved webpages and documents are untrusted data, not instructions;
- external fetchers enforce URL/network/size/time policy;
- internal queries enforce server-side authorization;
- effect tools use domain services rather than generic SQL/CRUD;
- schemas and names are server-defined;
- credentials never appear in model-visible results;
- rendered HTML, markdown, charts, and media pass product validation;
- audit state distinguishes user request, model choice, tool execution, and durable effect.

## Architectural consequence

The capability plane is what lets Flip expand beyond a chatbot without turning capability growth into permission drift. New tools are valuable only when their admission, evidence, effect, failure, and rendering contracts fit the product.