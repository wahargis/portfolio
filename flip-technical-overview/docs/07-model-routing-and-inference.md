# 07 — Model Routing as Execution Policy

## Product semantics must survive a model swap

Flip uses models for several different jobs: low-latency participation, evidence-heavy research, structured curation, document interpretation, and artifact continuation. Those jobs should not be defined by one provider’s API or one currently preferred model.

The product therefore owns a stable inference contract and treats route selection as execution policy.

<img src="../diagrams/model-routing-audit.svg" alt="Model routing and provider isolation" width="900" />

## Responsibility is deliberately layered

| Layer | Responsibility |
|---|---|
| **Flip runtime** | Identity, product context, admitted tools, evidence, terminal behavior, persistence. |
| **Route policy** | Select a compatible model/provider/local endpoint for this surface and deployment policy. |
| **Provider adapter** | Translate request, stream, tool-call, finish, and error behavior. |
| **Inference service** | Keep the model process or hosted capacity healthy and available. |
| **Model** | Reason, select among admitted tools, and compose content. |

This separation prevents a provider change from silently changing who the AI participant is, what it may read, or which product effects it may perform.

## Routes are selected by the work, not one global ranking

A useful route decision asks what the current surface requires:

- A fast conversational reply may prioritize latency and reliable direct composition.
- A research reply needs stable structured tool calls, sufficient working context, and good evidence use.
- Conversation curation values schema and plan validity more than personality.
- A document or vision task needs the required modality or a compatible dedicated tool.
- A privacy-sensitive room may require local inference even when a hosted route is stronger.

The same configured AI identity can use different execution routes without becoming a different participant. Persona, role, tone, permissions, and attribution are product state; the model is a replaceable resource used to realize them.

## The normalized request is a product-owned envelope

The runtime supplies selected context, product instructions, admitted schemas, output constraints, deadline, and the terminal-composition contract. The adapter converts that envelope to the provider’s shape.

Providers differ materially in streaming, tool-call encoding, forced-tool support, finish reasons, reasoning fields, token accounting, and error behavior. Adapters normalize those differences into the outcomes the runtime understands:

```text
assistant content
validated tool calls
finish state
usage / latency evidence
structured provider failure
```

Compatibility logic belongs at this boundary. Product contexts should not contain branches for every provider’s tool JSON or streaming protocol.

## Local and hosted inference remain interchangeable at the product boundary

A HomeCloud-hosted endpoint can provide data locality, fixed hardware economics, version/quantization control, and independence from one hosted vendor. It also introduces finite slots, model load time, GPU/process health, context-memory tradeoffs, and operator responsibility.

Flip consumes that endpoint through the same route contract. HomeCloud decides how local capacity is scheduled; Flip retains identity, permissions, tools, evidence, and persistence.

A local outage is therefore an AI-capability failure, not a corruption of chat or forum state. The product can queue, choose a policy-compatible route, or fail visibly while ordinary collaboration remains available.

## Fallback is constrained by the original contract

Fallback is not “try any model until something answers.” A replacement route must still satisfy modality, tool behavior, privacy, context, and terminal-composition requirements.

A valid fallback may change latency or cost. It must not:

- send local-only context to a remote provider contrary to policy;
- select a model unable to execute the admitted tools;
- lose citations or artifacts already gathered in earlier rounds;
- duplicate an effect that has already committed;
- convert an authentication/configuration failure into indefinite retry;
- persist provider protocol as user content.

When a provider fails after tools have already collected evidence, the runtime can preserve that evidence and attempt one bounded terminal composition through a compatible path.

## Route health and capability are different questions

An endpoint can be reachable but unsuitable. Route admission should distinguish:

- operational health and available capacity;
- support for required input/output modalities;
- structured tool-call reliability;
- terminal drafting compatibility;
- context and output limits under realistic latency;
- deployment privacy and cost policy.

A route that writes fluent prose but mishandles product actions is not healthy for an action-heavy surface.

## Context budgets are deployment decisions

The advertised context window is not the working budget. Conversation history, retrieved evidence, document passages, artifact metadata, tool schemas, and terminal output all compete for capacity. Hosted latency or local GPU memory can make the useful budget much smaller than the model maximum.

Flip therefore selects context before routing and reserves space for the final product response. A larger endpoint can improve capacity; it does not replace retrieval, relevance selection, or compaction discipline.

## Evaluation is surface-specific

A route is evaluated against the work it may receive:

- Does it select and call the admitted tools correctly?
- Does it produce a valid terminal answer rather than protocol leakage?
- Does it use citations near the claims they support?
- Does it preserve uncertainty and source disagreement?
- Does it produce valid curation plans?
- Does it continue honestly after tool or provider failure?
- Does it meet latency and capacity expectations for the deployment?

Results should be tied to model, route, adapter, configuration, and date. There is no useful global statement that one model is simply “the Flip model.”

## Audit boundary

Operational records can correlate the durable reply or artifact with the route, timing, usage where available, tool activity, terminal reason, and failure class. They should not retain credentials, unnecessary private content, hidden provider reasoning, or cross-community context.

## Architectural consequence

Models are execution components inside a product-owned runtime. Flip can adopt better models, local capacity, or new providers without surrendering the semantics that make the AI participant safe, attributable, and useful.