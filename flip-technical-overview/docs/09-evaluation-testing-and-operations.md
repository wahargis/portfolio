# 09 — Quality, Evaluation, and Operations

## Quality model

Flip needs several forms of evidence because a passing model response does not prove product correctness.

<img src="../diagrams/ci-quality-gates.svg" alt="Flip quality gates" width="900" />

| Evidence layer | Proves |
|---|---|
| **Domain unit tests** | Authorization, validation, state transitions, search, and pure logic. |
| **Database/integration tests** | Transactions, constraints, migrations, Oban uniqueness/retry, cross-context relationships. |
| **AI runtime contract tests** | Triggering, catalog selection, argument parsing, tool failures, finish/repair behavior, persistence. |
| **Provider adapter tests** | Request/response normalization, streaming, tool-call shape, compatibility fallbacks. |
| **Client tests** | State reducers, optimistic reconciliation, sync handling, rendering, native adapters. |
| **End-to-end scenarios** | One user-visible workflow across HTTP, jobs, database, sync/channels, and UI. |
| **Security tests** | Cross-community retrieval denial, URL/fetch policy, sanitization, secret redaction, admin boundaries. |
| **Evaluation corpora** | Relevance, citation support, curation validity, tool selection, and route-specific behavior. |
| **Operational telemetry** | Latency, error classes, retries, queue/capacity, terminal outcomes, recovery. |

No one layer substitutes for the others.

## Test the deterministic shell first

The highest-value tests assert code-owned invariants:

- an actor cannot search a community they cannot read;
- missing trusted scope returns no internal results;
- a tool exception cannot kill the reply job;
- a duplicate trigger cannot create a duplicate reply;
- a curation plan cannot reference a source outside its eligible set;
- user-authored source identity survives synthesis;
- an invalid citation identity cannot render as supported evidence;
- provider protocol markup cannot be persisted as a normal reply;
- client optimism converges with canonical transaction identity;
- a failed transaction cannot publish a phantom realtime update;
- an artifact continuation runs once per intended terminal event.

These remain valid across model upgrades.

## Model-behavior evaluation

Model-dependent behavior requires representative, dated cases rather than only unit fixtures.

### AI reply corpus

Include cases for:

- direct answer without tools;
- external search and source reading;
- conflicting sources;
- internal authorized retrieval;
- internal denied retrieval;
- document/PDF evidence;
- structured data and chart;
- asynchronous artifact continuation;
- tool failure;
- provider failure after evidence collection;
- terminal protocol leakage;
- insufficient evidence and honest uncertainty.

### Curation corpus

Include:

- one coherent topic;
- several interleaved topics;
- duplicate material already represented in a forum;
- participant correction;
- messages with reply dependencies;
- excluded/private material;
- invalid destination;
- low-signal conversation;
- content requiring manual review.

Evaluation can check structural plan validity, source coverage, authorship preservation, destination quality, duplicate rate, and reviewer correction rate.

### Route matrix

Each candidate route is assessed against the surfaces it may serve. A fast model that is unreliable at tool calls may still be useful for non-tool classification; a strong long-context route may be too slow for ordinary chat.

## Citation evaluation

A citation evaluation distinguishes:

1. citation identity exists;
2. source was actually retrieved;
3. selected passage appears in the source;
4. passage is relevant to the nearby claim;
5. claim does not exceed what the passage supports;
6. source type/date are represented honestly;
7. inaccessible or disputed source state is visible.

The first three are substantially deterministic. The latter require semantic evaluation and sometimes human review.

## Client synchronization tests

Realtime correctness should include adversarial ordering:

- mutation response before sync event;
- sync event before mutation response;
- duplicate event;
- reconnect during pending mutation;
- permission revoked while cached;
- two devices editing/reacting;
- background job updates while client sleeps;
- channel loss while durable sync continues;
- local clock skew;
- server migration/shape change.

The expected result is convergence, not one exact packet order.

## Job and recovery tests

Oban workflows should cover:

- uniqueness under simultaneous enqueue;
- retry after transient failure;
- terminal state not retried;
- partial synthesis/linkback failure;
- worker crash during tool execution;
- continuation deduplication;
- queue cancellation or shutdown;
- stale/pending artifact repair;
- backoff behavior without multi-hour hidden stalls.

Tests should assert product state, not merely that the worker function returned `:ok`.

## Security regression suite

Minimum high-value cases:

- cross-community chat/forum search;
- forged actor/community identifiers in tool arguments;
- missing authorization scope;
- private source included in a public synthesized artifact;
- server-side request forgery;
- malicious webpage/document prompt injection;
- unsafe rendered HTML/markdown;
- credential-like text in logs or source ledger;
- unauthorized platform action;
- stale client capability after permission change;
- admin setting manipulated through AI content.

## CI gates

A practical CI pipeline can include:

```text
format
  -> compile with warnings as errors
  -> dependency consistency
  -> database create/migrate
  -> server tests
  -> client tests/type checks/build
  -> artifact/diagram/docs checks
  -> dependency advisory checks
  -> container/release build
  -> selected integration scenarios
```

Large model/provider evaluations can run on a separate scheduled or pre-release track because they are slower and non-deterministic. Their results should still be versioned and tied to the route/model/configuration under test.

## Observability

### Product metrics

- active rooms/threads;
- message and forum participation;
- search use and successful navigation;
- synthesis creation/merge/correction;
- source-link traversal;
- AI invocation and response completion;
- artifact lifecycle.

### Runtime metrics

- queue wait and execution time;
- model latency and terminal reason;
- tool count, duration, and failure class;
- citation/artifact counts;
- retry and repair rate;
- provider circuit state;
- internal retrieval denial/missing-scope events;
- sync lag and reconnects.

### Quality signals

- participant feedback;
- recuration/manual-review rate;
- citation validation failure;
- unsupported-claim review;
- duplicate durable threads;
- AI reply deletion/correction;
- artifact abandonment;
- route-specific evaluation drift.

Metrics should be sliced by surface and route without exposing private content.

## Operational failure domains

| Failure domain | Isolation expectation |
|---|---|
| Hosted/local model endpoint | AI feature degrades; chat/forum transactions remain available. |
| One tool provider | That capability fails honestly; reply may continue with existing evidence. |
| Oban queue backlog | Durable jobs wait; synchronous product state remains coherent. |
| Electric/sync path | Clients can fall back/reconnect; server remains canonical. |
| Channel/PubSub interruption | Ephemeral events are lost; durable records recover. |
| Media provider | Artifact remains pending/failed; source conversation remains intact. |
| One Phoenix process | BEAM supervision restarts it without corrupting durable state. |
| Database unavailable | Reject/queue writes; do not publish non-durable success. |

## Deployment checks

Operational readiness should verify:

- schema migration compatibility;
- required feature/provider configuration;
- queue and schedule admission;
- database backup/restore;
- credential separation;
- route health and capability;
- sync/channel behavior;
- artifact storage access;
- rollback plan;
- observability and alert paths.

These checks belong in deployment automation or runbooks. They should not dominate the public product case study.

## Evidence discipline

Claims such as “production-ready,” “zero hallucinations,” “supports N users,” or “novel” require dated, reproducible evidence. The portfolio instead documents the architecture, deterministic contracts, representative evaluation methods, and known limitations.
