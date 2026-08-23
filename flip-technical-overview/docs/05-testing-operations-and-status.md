# Flip Testing, Operations, and Current Status

Flip's quality model covers product invariants, asynchronous workflows, AI execution, provider compatibility, and client convergence. A useful model response is not sufficient evidence that the surrounding product behaved correctly; authorization, persistence, delivery, recovery, and provenance must also be tested.

## Deterministic product tests

The largest class of tests covers behavior that should not depend on model judgment.

### Domain and authorization invariants

Tests verify conditions such as:

- room and forum access follows current membership and role state;
- cross-context reads preserve source-derived restrictions;
- AI tools use the triggering actor's product scope;
- deleted, inaccessible, or out-of-room content is excluded from context;
- curation and AI authorship remain distinguishable;
- citations and artifacts remain attached to eligible activities and messages;
- optional capabilities are not exposed when their backing configuration is absent.

These tests operate on product records and domain functions rather than evaluating generated prose.

### Workflow state machines

Synthesis, AI replies, media work, and delivery jobs are tested through their durable transitions:

- creation and enqueue occur together;
- work can be claimed only from eligible state;
- retries do not duplicate terminal product effects;
- stale or orphaned work can be detected and recovered;
- successful completion records the expected result and lineage;
- terminal failure remains queryable and does not masquerade as success;
- continuation uniqueness prevents competing delivery turns over one artifact or chain.

### Tool and effect contracts

Tool tests cover schema validation, argument sanitization, actor scope, effect authority, timeout classification, normalized results, audit persistence, citation creation, and artifact creation. Denied effects must remain denied in both durable state and the final response path.

### Realtime and synchronization behavior

Client tests exercise optimistic mutation identity, committed transaction reconciliation, reconnect behavior, capability negotiation, version compatibility, and the split between durable synchronized rows and ephemeral Channel events.

A new interface is not considered correct solely because it can display data. It must preserve the same command, authorization, lifecycle, and failure semantics as existing clients.

## AI-route evaluation

Model routes are evaluated against the behavior required by each product surface.

Relevant dimensions include:

- instruction and role adherence;
- tool-call schema correctness;
- multi-round tool use;
- terminal composition after bounds or provider failure;
- citation placement and evidence use;
- context-length and truncation behavior;
- output validation and repair;
- game or workflow-specific structured output;
- latency and deadline compatibility;
- compatibility with forced or unforced tool choice;
- artifact and continuation protocol handling.

A route can be suitable for ordinary conversation and unsuitable for a tool-heavy document or media workflow. Model selection therefore remains an execution policy informed by surface-specific tests rather than a single global quality ranking.

## Provider compatibility tests

Provider adapters are tested for request and response differences that can change product behavior:

- streaming and non-streaming responses;
- native and OpenAI-compatible tool formats;
- thinking or reasoning fields;
- tool-choice restrictions;
- usage reporting;
- context and output limits;
- timeout and cancellation behavior;
- provider-specific error payloads;
- credential and account rejection;
- malformed or partial protocol output.

Compatibility fallback is bounded and explicit. A provider-shape retry should not become an infinite sequence of slightly different requests, nor should a credential rejection be presented as a transient network fault.

## End-to-end cases

End-to-end testing combines several layers around a product outcome.

### Conversational AI

A case can create an actor, community, room, conversation, explicit trigger, admitted tool set, model response, tool result, citation, and final message. The assertions cover actor scope, context selection, activity records, final publication, and client delivery.

### Conversation curation

A case can select source messages, create and claim a synthesis run, produce a forum thread, attach source relationships, verify effective audience, and exercise feedback or retry.

### Long-running artifact

A case can create a pending artifact, schedule work, simulate terminal provider success or failure, deduplicate callbacks, deliver the result, and reconcile the client view.

### Client convergence

A case can issue an optimistic native mutation, commit the server transaction, receive Electric delivery, and verify that the local state converges without duplication. Rejection cases confirm that unauthorized or invalid tentative state is removed or marked failed.

## Operational telemetry

Operations need visibility into the product and the agent runtime without relying on model-authored explanations.

Representative telemetry and query surfaces include:

- Oban queue depth, attempt state, latency, and terminal outcome;
- AI activity route, duration, rounds, usage, finish reason, and recovery classification;
- tool-call category, duration, result, denial, and timeout;
- artifact queue, provider state, completion, failure, and delivery latency;
- synthesis claim age, retry, completion, and stale-work recovery;
- client synchronization and mutation-reconciliation errors;
- provider health, request failure class, and route fallback;
- application process health, database state, and endpoint readiness.

Detailed administrative traces remain separate from reader-facing source ledgers. The operational system can retain enough information to diagnose a failed turn without disclosing hidden context or sensitive arguments to ordinary community members.

## Recovery operations

The runtime includes explicit operations for conditions that cannot be resolved inside the original request:

- retry available Oban jobs under bounded policy;
- identify orphaned AI activities or media deliveries;
- reconcile a terminal provider job with an incomplete product record;
- recover stale synthesis claims;
- publish an honest terminal disclosure when work cannot continue;
- prune or retain audit state according to lifecycle policy;
- disable an unhealthy provider or capability while ordinary community functions remain available.

Recovery changes durable state and is therefore tested and audited like any other product effect.

## Public synthetic environment

The synthetic environment provides a safe deployment for product and architecture inspection. It uses generated accounts, communities, messages, documents, media, AI activities, and failures. It does not mirror production data, credentials, private messages, moderation authority, or security-sensitive configuration.

The environment can demonstrate:

- live chat and forum interaction;
- explicit AI invocation and visible activity state;
- tool and citation rendering;
- pending and terminal artifacts;
- conversation-to-forum synthesis;
- web and native synchronization behavior;
- representative provider and workflow failures.

Its purpose is to expose actual application behavior while preserving a strict authority and data boundary from the product deployment.

## Implemented system scope

The current codebase includes substantial behavior in these areas:

- accounts, communities, membership, roles, and moderation;
- live chat, replies, reactions, polls, files, presence, and search;
- forums, nested discussion, voting, bookmarks, tags, and provenance;
- explicit AI participation, tools, retrieval, citations, and artifacts;
- asynchronous conversation curation;
- image, video, audio, and other media workflows;
- web, desktop, mobile, and PWA clients;
- capability negotiation, synchronization, notification, billing, usage, and operations support.

The portfolio concentrates on the load-bearing architecture rather than attempting to inventory every feature or framework.

## Current pressure points and boundaries

- Large agent and media workers require continued decomposition so provider compatibility, terminal delivery, and recovery remain independently testable.
- Surface-specific model evaluation must continue as routes and providers change; interface compatibility does not imply equivalent behavior.
- Client synchronization requires ongoing schema and protocol discipline as more product state becomes available offline.
- Public demonstrations must remain synthetic and cannot expose production community state or administrative authority.
- Local and hosted inference can share an application contract, but capacity, latency, privacy, and failure characteristics remain materially different.
- Tool breadth must not outrun actor authorization, effect policy, auditability, or client rendering support.

## Selected private implementation paths

| Path | Responsibility |
|---|---|
| `test/flip/` | Domain, authorization, workflow, AI, tool, artifact, and persistence tests. |
| `test/flip_web/` | Web, API, LiveView, Channel, and authorization integration tests. |
| `flip-client/src/` and client tests | React client state, synchronization, optimistic mutation, and rendering behavior. |
| `lib/flip/synthesis/ai_reply_telemetry.ex` | AI-turn telemetry and operational events. |
| `lib/flip/llm/call_audit.ex` and `audit_pruner.ex` | Provider-call audit state and retention. |
| `lib/flip/llm/orphaned_activity_reaper.ex` | Detection and recovery of incomplete AI activity. |
| `lib/flip/metrics.ex` and `lib/flip/metrics/` | Application and product metrics. |
| `lib/flip/application.ex` | Supervision, queues, provider clients, and runtime health boundaries. |
| `.github/workflows/` | Automated test, static-analysis, and release checks. |

[← Flip case study](../README.md)
