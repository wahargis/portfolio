# Agent runtime

Flip's agent runtime converts a product event into a bounded model execution and a durable product outcome. The runtime owns admission, context, route and capability selection, tool rounds, terminal validation, activity records, continuation, and failure state.

![Agent execution sequence](../diagrams/agent-execution-sequence.svg)

## Turn identity

Each execution has a durable identity associated with:

- the invoking human or system actor;
- the AI identity that will be attributed in the product;
- the community and origin object;
- the product surface and feature configuration;
- the model and provider route;
- the parent activity or asynchronous artifact when this is a continuation;
- request and effect identities used for deduplication.

The provider's conversation identifier can be stored as metadata but is not the primary product identity.

## Admission

Admission occurs before protected context is loaded or a provider request is made.

The server verifies:

1. the invoking actor is authenticated where required;
2. the actor can read and act in the origin surface;
3. the selected AI identity is enabled for that surface;
4. the trigger is valid and has not already produced the intended run;
5. required services and routes are configured and healthy enough for the request;
6. workload, rate, and concurrency limits permit execution;
7. the operation is not a stale continuation or duplicate terminal event.

A rejected admission produces a product-appropriate error or no-op. It does not expose hidden context or make a provider call only to discard the result later.

## Context construction

Context is assembled from typed sources under a working budget.

### Product conversation

Recent messages, selected references, and same-room reply ancestry provide local continuity. The runtime excludes product machinery and content outside the current scope. It can add room or AI-identity configuration where that information is appropriate for the participant.

### Internal retrieval

Product search and direct reads receive trusted actor and community scope. Results include stable product identities and visibility-relevant metadata so later reads and citations can be checked again.

### External sources

Search identifies candidate sources. Direct source tools retrieve pages, documents, or passages used by the turn. The runtime stores source and citation records instead of leaving URLs only in provider text.

### Documents and data

Uploaded or selected files, extracted passages, structured tables, and analysis results are represented as application objects. Context includes the selected material and references to durable results, not an unbounded copy of every document or dataset.

### Prior activity and artifacts

A continuation can load the completed artifact, relevant prior tool state, and the original product scope. It does not need to replay the entire provider exchange that requested the operation.

## Route and capability selection

The runtime selects a route using workload requirements such as:

- text, vision, document, audio, image, or video modality;
- context budget;
- structured-output and tool-call behavior;
- required local or hosted execution;
- privacy and data-handling constraints;
- latency, throughput, and cost policy;
- current provider or local endpoint health;
- evaluation status for the product surface.

The capability catalog is computed for the specific turn. Tools are included only when they are available, compatible with the route, allowed for the product surface, and safe under the current actor and origin scope.

## Working rounds

A turn can contain several model rounds.

```text
selected context and tool catalog
  -> model response
     -> terminal draft
     -> one or more tool requests
     -> refusal or protocol error
  -> validate tool requests
  -> execute admitted capabilities
  -> append structured results and continue
  -> stop on terminal output or bounded failure
```

The runtime tracks round count, deadlines, accumulated tool state, unresolved asynchronous work, and terminal status. Tool output is treated as structured runtime input. Provider text that resembles a tool result does not receive the authority of an executed application capability.

## Tool-call processing

A tool call passes through:

1. schema and type validation;
2. capability lookup for the current turn;
3. trusted actor, community, origin, and workspace injection where required;
4. authorization and domain validation;
5. timeout, concurrency, and retry classification;
6. tool execution through the owning application service;
7. durable result or effect identity creation where applicable;
8. normalized result returned to the model round;
9. activity, source, artifact, and failure recording.

Read-only retrieval, synchronous effects, and long-running artifacts use different lifecycle paths. An effectful tool is not automatically retried unless its identity and domain behavior make retry safe.

## Terminal response

A terminal result is validated before publication.

Validation can include:

- separation of internal working protocol from the user-visible reply;
- required response structure for the product surface;
- citation references that correspond to stored sources;
- artifact identifiers that exist and are visible to the user;
- absence of unresolved required tool state;
- output and size limits;
- product-specific restrictions on actions, links, or attachments;
- final access check for the target object.

After validation, the application commits the AI activity, user-visible reply or result, references, and terminal status. Publication and realtime delivery occur from committed state.

## Asynchronous capability path

Some tools create pending artifacts or provider jobs instead of a final result.

The initial turn receives a durable pending object and can tell the user that work has started. A worker or callback updates the object through provider attempts and terminal state. Completion or failure can create one continuation event associated with the original activity and product scope.

The continuation is admitted as a new bounded execution. It can present the artifact, perform a later stage, or explain failure. Duplicate callbacks and repeated terminal events do not create repeated replies because continuation identity is stored.

## Activity and source records

The runtime stores an administrative activity record with the information needed for diagnosis and operation. A separate reader-facing projection can show relevant source and tool status without revealing private provider arguments or internal errors.

Activity state can include:

- start, progress, terminal, and retry timestamps;
- selected route and provider attempt;
- input and output accounting where available;
- tool names, state, duration, result identity, and failure classification;
- sources and citations used by the final reply;
- artifacts and product effects created;
- parent and continuation relationships;
- terminal success, refusal, cancellation, timeout, or failure.

## Cancellation, retry, and recovery

Cancellation is applied to application state first. The runtime then requests provider or worker cancellation where supported and records whether the external operation confirmed termination.

Retries depend on failure class:

- transient provider or transport failures may retry within the route policy;
- invalid model protocol may use repair or a compatible route;
- deterministic authorization or validation failures do not retry as provider work;
- effectful tool failures retry only when the operation is idempotent or can be safely resumed;
- long-running artifacts retry from their durable stage state;
- a stale or duplicate continuation is rejected.

A process restart does not erase the activity, job, artifact, or terminal state. Recovery selects incomplete durable work and resumes or fails it according to the relevant lifecycle.

[Previous: System architecture](02-system-architecture.md) · [Next: Retrieval, evidence, and tools](04-retrieval-search-and-tools.md)
