# Flip AI Participant Runtime

Flip's conversational AI is implemented as a durable product workflow. A visible user action starts the turn; product code determines the eligible context and capabilities; an asynchronous worker performs model and tool rounds; and the result is committed as an identifiable chat message with separate execution evidence.

## Turn lifecycle

```text
trigger
  -> admission
  -> context construction
  -> capability and effect admission
  -> model/tool rounds
  -> terminal composition
  -> evidence and artifact persistence
  -> chat publication
  -> optional asynchronous continuation
```

## Trigger detection and admission

An AI turn starts from a product event such as a mention of Flip or a reply to an AI message. Trigger detection determines:

- the triggering message and actor;
- the room and community;
- whether AI participation is enabled for that surface;
- whether the event is eligible for a new reply;
- whether equivalent work is already queued or executing;
- which room-specific configuration applies.

The triggering user remains part of the authorization context. A queued worker does not become a platform administrator merely because it executes outside the original request process.

Admission creates or identifies the durable work required by the turn and schedules it through Oban. Unique job and activity identities collapse duplicate enqueue attempts and allow retries to inspect prior state.

## Context construction

Context is selected from product state, not copied from every record available to the application.

### Room-scoped conversation

The runtime can include a bounded window of recent eligible messages from the current room. System machinery, deleted or inaccessible content, and content outside the authorized room boundary are excluded.

### Reply ancestry

The reply chain can be followed to recover the immediate conversational thread. Ancestry remains bounded and same-room; it is not used as an implicit cross-room retrieval mechanism.

### Room briefing and product state

A room may define a briefing or configuration relevant to AI participation. The runtime can also supply typed state required by a specific surface, such as game state, poll state, an artifact request, or a continuation record.

### Context budgets

The provider route and model determine the available context window. Product code allocates that window among instructions, conversation, tool definitions, retrieved evidence, prior tool results, and terminal-composition reserve. Large context support does not eliminate the need to select relevant state or preserve room boundaries.

## Capability and effect admission

Tools are assembled for the current turn from product policy and route capability.

A tool definition includes:

- a stable operation name;
- a typed argument schema;
- the actor and product scope required to execute it;
- timeout and retry behavior;
- a normalized result contract;
- whether it reads state, creates an artifact, or performs another durable effect.

The model can propose a tool call. Product code validates the schema and then applies effect authority before execution. Tool availability may differ between ordinary conversation, document work, media generation, games, moderation-adjacent workflows, or administrative surfaces.

This keeps capability selection explicit. A provider that supports function calling does not automatically receive every operation implemented by the application.

## Durable model and tool loop

`Flip.Synthesis.AiReplyWorker` executes the turn as an Oban worker. It maintains the working message history, admitted tools, route configuration, elapsed time, round state, prior results, draft state, and recovery state across model calls.

### Model rounds

A round can produce:

- a terminal answer;
- one or more tool calls;
- a draft update;
- provider protocol or validation failure;
- a request that requires a specialized continuation path.

The worker records model and tool activity separately from the final message. This allows operations and audit code to distinguish a fluent answer from the calls, evidence, failures, and route behavior that produced it.

### Tool execution

Tool calls are parsed and sanitized before dispatch. Independent calls may run concurrently where their effects and resource requirements permit it. Results are normalized before returning to the model and persisted when they carry evidence or durable product state.

Errors remain typed. A malformed call, denied effect, timeout, provider failure, missing document, or terminal media failure should not all become the same text string.

### Round and deadline controls

The turn has product-configured round and deadline policy. Reaching a bound does not immediately emit a canned timeout response. The worker enters a terminal-composition mode that removes ordinary tool access, supplies the evidence already gathered, and requires the model to write the complete user-facing answer into the durable draft channel.

A finish reserve protects enough time for that final composition after a slow tool call. Tool timeouts are clamped against the remaining turn window so one operation cannot consume the entire terminal budget unnoticed.

## Terminal composition and output validation

The terminal response passes through product checks before publication.

Validation detects conditions such as:

- blank output;
- leaked tool or provider protocol markup;
- internal template text;
- an unusable heuristic refusal;
- malformed citations or artifact references;
- missing required game or workflow output.

Where safe context remains available, the worker can spend one bounded model-authored repair turn. If a valid response still cannot be produced, the application publishes or records an honest terminal disclosure rather than presenting hidden protocol text or silently marking the turn successful.

The final message is created through the chat domain with the system AI identity, room scope, and reply relationship to the triggering content.

## Evidence, citations, and activity

The AI message and the execution record are related but distinct.

### Administrative activity

Detailed activity can include route, provider, model, timing, usage, rounds, tool calls, sanitized arguments, errors, retries, and terminal classification. This supports operations, debugging, and policy evaluation.

### Reader-facing source ledger

A public projection can show citations, source status, tool categories, warnings, and artifact outcomes that help a reader assess the answer. It excludes credentials, sensitive tool arguments, private provider metadata, and unrestricted internal traces.

### Durable citations

Citations are stored as product records linked to the activity or response. The final message embeds stable citation references at the relevant claims. The evidence therefore remains resolvable after the model context and external request have ended.

## Artifacts and asynchronous continuation

Some operations cannot complete within one conversational request. Image, video, document, and other artifact workflows use durable identities and explicit lifecycle state.

A turn can create a pending artifact and schedule external or local work. The resulting job records completion or failure against the same artifact. A unique continuation job may then:

- update the original message or artifact preview;
- add the terminal result to the conversation;
- ask the model to compose a bounded follow-up from the terminal facts;
- continue a multi-step media chain without racing another step in the same chain.

Uniqueness is keyed to the durable job, response, or chain identity so simultaneous provider callbacks and retries converge on one delivery path.

## Provider and failure recovery

Provider failures are classified rather than retried uniformly.

- Transient network and service failures can use bounded backoff.
- Request-shape incompatibility may retry through a provider-specific compatibility path.
- Credential, account, or billing rejection can trigger one configuration refresh and then an explicit terminal result.
- A provider that dies after tool work can enter terminal composition using already gathered evidence.
- A worker that disappears without terminal delivery can be recovered through Oban retry and orphan-recovery state.

The worker's retry policy is idempotent with respect to durable publication. A retry finding an existing terminal message or disclosure exits without posting a second result.

## Selected private implementation paths

| Path | Responsibility |
|---|---|
| `lib/flip/synthesis/ai_reply_detector.ex` | Trigger recognition, actor and room admission, recent context, reply ancestry, and room briefing. |
| `lib/flip/synthesis/ai_reply_enqueuer.ex` | Durable enqueue and uniqueness behavior for conversational turns. |
| `lib/flip/synthesis/ai_reply_worker.ex` | Model/tool loop, bounds, finish mode, output validation, repair, publication, and continuation. |
| `lib/flip/synthesis/action_authority.ex` | Authorization of model-proposed effects. |
| `lib/flip/synthesis/ai_reply_link_validator.ex` | Validation of links and references emitted in AI replies. |
| `lib/flip/synthesis/ai_reply_telemetry.ex` | Turn-level operational telemetry. |
| `lib/flip/llm/activity.ex` and `activities.ex` | Durable AI activity state and queries. |
| `lib/flip/llm/tool_call.ex` and `tool_call_audit.ex` | Tool-call lifecycle and audit projection. |
| `lib/flip/llm/source_ledger.ex` | Reader-facing evidence projection. |
| `lib/flip/synthesis/artifact.ex`, `artifact_preflight.ex`, and `artifact_wire.ex` | Typed artifact creation, admission, and delivery references. |

[← Flip case study](../README.md)
