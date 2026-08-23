# Synthetic technical environment

The synthetic environment exercises Flip's product and agent contracts without using product data, private credentials, or administrative authority. It is intended for architecture review, deterministic testing, and failure analysis.

It is not a copy of production. It uses synthetic identities, communities, rooms, forums, messages, documents, sources, artifacts, model outputs, and provider events under a separate namespace and data store.

## Environment boundary

| Shared with the product architecture | Kept separate |
|---|---|
| Domain schemas and state transitions | Product accounts, communities, messages, and media |
| Authorization and origin rules | Product authentication and administrator sessions |
| Agent admission, context, tools, and terminal validation | Product provider credentials and route configuration |
| Job, run, artifact, retry, and continuation behavior | Product queues, callbacks, storage buckets, and deployment state |
| Realtime and durable client contracts | Product push credentials, deep links, and client release channels |
| Deterministic and provider-fixture evaluation paths | Product telemetry, billing, and user activity |

The environment must fail closed when a product-only dependency is requested. It should not proxy unknown reads or effects to a product deployment.

## Synthetic data

Fixtures use clearly non-production identifiers and content. The fixture set can include:

- human and AI identities with different community and room membership;
- public, private, and restricted rooms and forums;
- conversation threads with replies, media references, and source-selection candidates;
- documents, web-source fixtures, data tables, charts, images, and video job records;
- curation destinations and source relationships;
- provider streams, tool calls, timeouts, malformed output, and asynchronous completion events;
- web and native client identities with controlled capability metadata.

No fixture should be derived from copied product content or a product database export.

## Scenario set

### Direct AI reply

A member invokes an AI identity in an allowed room. The scenario verifies actor and origin resolution, bounded context, route and tool admission, activity state, terminal reply validation, and client delivery.

Variants cover unavailable AI identity, revoked room access, duplicate trigger, invalid tool request, provider failure, and a response that refers to a missing citation or artifact.

### Scoped internal retrieval

A user asks for information available in one permitted room while similar content exists in another restricted room. The scenario verifies that retrieval uses trusted actor and community scope and that the restricted record is not included or discoverable through model arguments.

### External research and citation

A request requires search, direct source reads, and a final cited response. The scenario verifies the separation of discovery from evidence reads, durable source records, citation validation, incomplete-source handling, and the reader-facing evidence projection.

### Product action

The model requests a typed product action such as a poll or forum operation. The scenario verifies target validation, idempotent effect identity, committed product state, returned tool result, and rejection of a target outside the actor's scope.

### Asynchronous artifact

A media or document operation creates a pending artifact, receives progress, completes or fails after the originating turn, and starts at most one continuation. The scenario verifies restart, duplicate provider events, retry, cancellation, missing output, and final client state.

### Conversation curation

Selected source messages enter a curation run. The scenario verifies destination planning, source author and message relationships, origin-aware access, partial stage failure, linkback, feedback, and recuration without duplicate publication.

### Client convergence

Web and native clients submit a command and receive a response, durable synchronization, and realtime updates in several orders. The scenario verifies canonical identity, optimistic reconciliation, reconnect, duplicate suppression, and removal of data after access revocation.

## Evidence produced by a scenario

A scenario should leave inspectable state rather than only a screenshot or generated answer. Depending on the path, this can include:

- product object and canonical identity;
- actor, AI identity, origin, and visibility inputs;
- AI activity and tool-call records;
- selected sources, citations, and artifacts;
- job or run lifecycle and retry state;
- continuation identity and deduplication state;
- client-visible durable and realtime events;
- deterministic assertions and provider-fixture evaluation output.

## Safety controls

- Separate database, queues, storage, credentials, hostnames, and callback routes.
- Synthetic-only account and community namespaces.
- No product administrator tokens or copied sessions.
- No provider key shared with a product environment unless the key is independently scoped and approved for the synthetic environment.
- Deny product hostnames, storage roots, and callback targets at configuration and network boundaries where practical.
- Redact or reject secret-shaped content in fixtures and exported evidence.
- Keep generated reports free of hidden prompts, credentials, private routes, and internal host identifiers.

## Relationship to the portfolio

The synthetic environment supports the public diagrams and technical descriptions by exercising the same contracts with non-sensitive data. It does not establish product scale, production reliability, or access to the private implementation repository.

[Back to the Flip case study](../README.md)
