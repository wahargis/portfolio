# 03 — Agent Runtime

## Purpose

Flip’s AI participant runtime turns a product event into one attributed, durable, evidence-aware product effect. It is not a general autonomous loop detached from the application.

<img src="../diagrams/agent-execution-sequence.svg" alt="AI reply lifecycle sequence" width="900" />

## Lifecycle

```text
product event
  -> eligibility and trigger detection
  -> unique durable job
  -> origin actor/community scope
  -> concurrency admission
  -> context and persona assembly
  -> capability catalog
  -> provider request
  -> zero or more tool rounds
  -> terminal draft composition
  -> output and citation validation
  -> durable reply/artifact persistence
  -> realtime publication
  -> telemetry and cleanup
```

Every stage has code-owned inputs and terminal behavior.

## 1. Trigger and eligibility

Triggers include direct mentions/replies, scheduled curation, forum enrichment, reactions, artifact terminal events, or specialized product events. The detector decides:

- whether the event is intended for an AI participant;
- which AI identity and surface apply;
- whether feature and room settings permit the behavior;
- whether an equivalent job already exists;
- what causal identifiers should be carried into the run.

The worker does not infer a new user request from arbitrary ambient traffic.

## 2. Durable job and deduplication

The job envelope carries stable product identifiers rather than serialized private content where possible. Uniqueness constraints prevent duplicate replies for one trigger. Specialized chains can use a chain-level key so simultaneous provider completions do not race into several continuation turns.

The durable job provides:

- retry identity;
- attempt count and backoff;
- terminal job state;
- a place to distinguish provider, tool, output, and persistence failures;
- visibility independent of a web process.

## 3. Authorization scope

The runtime derives the scope from product state:

```text
origin server/community
origin room or forum
invoking actor
AI participant identity
surface type
specialized object identity, when any
```

This scope follows the tool call into child execution tasks. It is not reconstructed from model arguments. Internal search without a valid origin scope fails closed.

## 4. Context assembly

Context is selected, not dumped. It can include:

- triggering message and reply chain;
- bounded recent room context;
- forum thread context;
- AI persona and community instructions;
- prior draft or continuation state;
- artifact/request state;
- tool-generated citations already minted;
- specialized domain state such as a game position.

Selection respects visibility and context budgets. A model’s large window does not authorize reading an entire community.

## 5. Capability catalog

The server builds a schema list from:

- current surface (`chat`, `forum`, `synthesis`, `personal`, `enrichment`, specialized lane);
- feature flags;
- actor/community permissions;
- available provider credentials/services;
- object context;
- safety and product rules.

Some surfaces use a replacement catalog, not merely a filtered general catalog. For example, a curation loop receives read/analysis tools and a terminal plan submission tool; a specialized game turn receives game and drafting capabilities rather than external research/media tools.

<img src="../diagrams/governed-tool-execution.svg" alt="Governed tool execution" width="900" />

## 6. Provider turn

The model client normalizes product intent into a provider-compatible request:

- messages and system context;
- tool schemas;
- output/token limits;
- model/provider route;
- optional forced tool choice for terminal composition;
- timeout and retry policy;
- provider-specific compatibility adjustments.

Provider-specific differences remain below the runtime contract. The product expects either an assistant message, validated tool calls, or a structured failure.

## 7. Tool execution

Each tool call is parsed and dispatched by name to a server handler. Execution occurs in isolated tasks with timeout and exception normalization.

The dispatcher returns a structured tool message or effect envelope. An unexpected raise, exit, or throw becomes an honest result explaining that no data was returned. The model is instructed not to retry blindly or fabricate.

For effectful tools, the dispatcher may return:

- model-visible content;
- a durable artifact or action identifier;
- intents that must be committed by the product layer;
- citation tokens;
- a continuation handle.

The model does not write product tables directly.

## 8. Iteration bounds

The loop can be constrained by:

- explicit round limit;
- wall-clock deadline;
- per-tool timeout classes;
- terminal-call reserve;
- provider output limit;
- concurrency semaphore;
- specialized repair budgets.

A deployment can configure generous bounds or an explicit no-round-limit sentinel while retaining a deadline. The key contract is that bounds converge on a terminal behavior rather than abruptly emitting a misleading canned answer.

## 9. Terminal composition

A working turn and a user-visible reply are different channels. The runtime reserves or forces a terminal composition step so the model cannot finish with only internal tool intent.

Terminal rules can require a dedicated drafting tool or a final prose-only call depending on provider compatibility. The final prompt reminds the model to use already gathered evidence, include citation identities, and avoid narrating internal mechanics.

## 10. Output validation and recovery

Before persistence, the runtime checks for conditions such as:

- blank output;
- leaked provider/tool protocol markup;
- hidden template syntax;
- invalid artifact or citation references;
- heuristic refusal where usable context exists;
- unsafe or structurally invalid rendering.

A bounded recovery turn can ask the model to rewrite from the safe context already gathered. If repair fails, the product persists an honest failure/disclosure state rather than infinite retries or protocol leakage.

## 11. Persistence

The final effect is committed through product contexts:

- AI identity and attribution;
- reply-to/source relationship;
- content type;
- citation and artifact associations;
- causal job/request identity;
- terminal state;
- relevant audit and telemetry records.

Only after persistence does the product publish the durable change to clients.

## 12. Continuation

Asynchronous media/document workflows may complete after the initiating AI turn. Their terminal event can enqueue one deduplicated continuation carrying durable identifiers. The next turn receives the artifact state and composes a product-context response.

Continuation is not a revived process. It is a new bounded turn linked to the prior request.

## Concurrency and backpressure

Separate mechanisms address separate contention:

- Oban queue concurrency controls worker throughput;
- per-community semaphores prevent one community from monopolizing reply capacity;
- provider/model routing limits protect endpoints;
- tool-specific queues protect long media work;
- database uniqueness prevents duplicate effects;
- client progress events do not imply extra execution capacity.

## Failure taxonomy

| Class | Example | Product response |
|---|---|---|
| **Eligibility** | feature disabled, not a valid trigger | no job/effect |
| **Authorization** | missing internal-search scope | empty authorized result, no leak |
| **Capacity** | concurrency unavailable | wait/retry under queue policy |
| **Provider** | timeout, rejected request shape | bounded provider retry/compatibility fallback |
| **Tool** | exception or timeout | model-visible honest failure result |
| **Evidence** | citation cannot be validated | omit/revise unsupported claim |
| **Output** | blank or protocol leak | bounded terminal repair |
| **Persistence** | transaction failure | no published phantom reply; retry if safe |
| **Continuation** | provider artifact fails | durable failed artifact plus honest follow-up |
| **Terminal** | repair exhausted | explicit failure/disclosure, no infinite loop |

## What the model controls

- which admitted tool is useful;
- query formulation;
- source interpretation;
- synthesis and answer content;
- whether more evidence is needed;
- selection among allowed product actions.

## What code controls

- identity, authorization, and scope;
- catalog membership;
- execution isolation and timeout;
- durable effect schema;
- job uniqueness and retries;
- provider credentials;
- citation/artifact identity;
- output validation;
- persistence;
- notification;
- terminal failure semantics.

That boundary is the runtime’s primary safety and reliability mechanism.
