# 07 — Model Routing and Inference

## Product semantics must survive model changes

Flip uses provider-compatible model boundaries so a product capability is not defined by one vendor’s request shape. Hosted providers, routing aggregators, and self-hosted OpenAI-compatible endpoints can supply inference, but the product retains authority over context, tools, citations, persistence, and user-visible behavior.

<img src="../diagrams/model-routing-audit.svg" alt="Model routing, circuit breaker, and audit flow" width="900" />

## Routing inputs

A route can depend on:

- AI participant/persona configuration;
- surface type: chat, forum, curation, enrichment, artifact continuation;
- required modalities or tool-call support;
- context and output requirements;
- provider availability and health;
- deployment policy;
- cost/latency class;
- local versus remote endpoint preference;
- compatibility constraints for forced tools, reasoning modes, or streaming.

The model should not select its own credentials or endpoint.

## Layered responsibility

| Layer | Owns |
|---|---|
| **Product runtime** | Trigger, actor scope, product context, admitted tools, terminal behavior, persistence. |
| **Route policy** | Model/provider choice, capability match, fallback order, local/remote preference. |
| **Provider adapter** | Request/response shape, streaming normalization, tool-call decoding, provider-specific compatibility. |
| **Inference service** | Model process health, batching/slots, GPU placement, throughput. |
| **Model** | Reasoning, tool selection among admitted options, answer/artifact content. |

A local HomeCloud endpoint participates at the inference-service layer; it does not own Flip permissions or tool admission.

## Provider-compatible request envelope

The runtime constructs a normalized envelope containing:

- system/product instructions;
- selected messages and context;
- admitted tool schemas;
- output/token limit;
- optional forced terminal tool;
- streaming callback or non-stream response;
- deadline;
- trace/correlation identity.

The adapter maps this to the provider without exposing provider quirks to product contexts.

## Response normalization

Providers can differ in:

- tool-call JSON shape;
- streaming delta format;
- finish reasons;
- reasoning/thinking fields;
- support for forced tool choice;
- treatment of empty tools;
- context/token accounting;
- error classes and status codes.

Adapters normalize these into:

```text
assistant content
validated tool calls
finish state
usage/latency metadata
structured provider error
```

The reply runtime then applies one lifecycle regardless of provider.

## Capability matching

A route should be admitted only when the selected endpoint supports the required surface.

| Requirement | Route implication |
|---|---|
| Tool-using research | Reliable structured tool calls and sufficient context. |
| Terminal drafting tool | Provider supports forced tool choice, or runtime selects a compatible fallback strategy. |
| Vision description | Multimodal input support or a dedicated vision tool. |
| Long document synthesis | Appropriate context plus retrieval/compaction; context alone is not enough. |
| Media generation | Usually a tool/provider workflow, not the chat model itself. |
| Local privacy preference | Select a compatible local endpoint; external tools may still create network egress. |
| Low-latency chat | Prefer a fast route with product-acceptable quality and tool behavior. |
| Conversation curation | Prefer reliable schema/plan production over conversational personality. |

## Persona versus model

Persona is product configuration:

- identity and attribution;
- role and tone;
- community-specific guidance;
- permitted product behavior;
- specialized surface instructions.

Model is an execution resource. Changing the model must not silently create a new identity or expand permissions. Several personas can share a route; one persona can use different routes based on capability and health.

## Local inference

A self-hosted endpoint can provide:

- data locality for model inputs;
- fixed hardware economics;
- control over model/version/quantization;
- provider independence;
- custom capacity allocation.

It also introduces:

- finite concurrency;
- model warmup/load time;
- GPU/process health;
- context-memory tradeoffs;
- operator responsibility;
- possible capability gaps versus hosted services.

Flip therefore consumes local inference through the same provider-compatible boundary and handles unavailability as an AI-feature degradation, not a core product database failure.

## Circuit and fallback behavior

A circuit breaker prevents repeated provider failure from multiplying latency and queue pressure. Routing policy can distinguish:

- transient timeout;
- provider request-shape incompatibility;
- rate/capacity rejection;
- authentication/configuration failure;
- invalid model output;
- local endpoint health failure.

Fallback should preserve the product contract. It must not:

- send private/local-only context to a remote provider contrary to policy;
- choose a model that cannot execute the admitted tools;
- duplicate an already completed product effect;
- erase citation/artifact state from earlier rounds;
- retry indefinitely.

## Tool and model coupling

Tool schemas are product-owned, but providers vary in how faithfully they invoke them. The runtime defends this boundary with:

- strict name/argument parsing;
- server-side allowlists;
- context-specific catalogs;
- repair for malformed calls where safe;
- provider-specific terminal strategy;
- protocol-leak detection;
- explicit tool failures;
- model-route evaluation against representative tool tasks.

A route that produces fluent prose but unreliable effect calls is unsuitable for an action-heavy surface.

## Context and output budgets

The route policy and runtime coordinate:

- selected conversation length;
- retrieved tool results;
- document excerpts;
- artifact metadata;
- maximum model output;
- terminal-call reserve;
- provider context limit;
- deadline.

The model’s advertised maximum context is not the working budget. Useful latency and local GPU memory can impose a lower deployment-specific limit.

## Audit and privacy

Provider audit records can include:

- route identity and model class;
- request/response timing;
- token/usage fields where available;
- tool-call count and terminal reason;
- error class;
- local versus remote route;
- correlation with durable reply/artifact.

They should exclude:

- API keys;
- raw private content unnecessary for operations;
- hidden provider reasoning;
- credentials embedded in tool results;
- cross-community context.

## Evaluation

Routes should be evaluated by surface, not one global benchmark:

- citation-bearing current-information replies;
- internal-search authorization and relevance;
- tool-call correctness;
- terminal composition reliability;
- conversation curation plan validity;
- artifact/media continuation;
- latency under realistic context;
- refusal and failure honesty;
- local endpoint capacity behavior.

## Failure scenarios

| Scenario | Expected behavior |
|---|---|
| Primary route unavailable before first turn | Choose policy-compatible fallback or fail visibly. |
| Provider fails after tools gathered evidence | Preserve evidence and attempt bounded terminal recovery. |
| Provider rejects forced tool shape | Use known compatibility strategy, not unbounded retries. |
| Local endpoint has no slot | Queue/backpressure or compatible fallback; do not bypass scheduler. |
| Fallback lacks required modality | Return capability error; do not pretend it can complete. |
| Remote route prohibited by privacy policy | Fail locally rather than exfiltrate context. |
| Model returns protocol markup | Reject/repair before persistence. |

## Architectural principle

Models are replaceable execution components. Product correctness must be expressed in the runtime, domain model, evidence layer, and tests so that changing a route does not silently change what Flip means.
