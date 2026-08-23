# Model routing and inference

Flip routes AI workloads to local or hosted model services through product-owned request and event contracts. Routes are selected by workload requirements and current operating state. A provider or model change does not alter product identity, access, tools, effects, or persistence.

![Model routing and audit](../diagrams/model-routing-audit.svg)

## Workload profiles

Different product surfaces require different model behavior.

| Workload | Typical requirements |
|---|---|
| **Realtime conversation** | Low latency, reliable conversational continuation, bounded context, selected product tools, and fast failure. |
| **Evidence-heavy research** | Larger working context, reliable tool use, source handling, document reading, and cited terminal output. |
| **Curation and structured planning** | Schema adherence, coverage of selected source material, destination planning, and stable retry behavior. |
| **Document and data work** | File or table context, analysis tools, structured results, chart or artifact references, and reproducible operations. |
| **Vision and multimodal analysis** | Image, video, OCR, or document modality support with defined size and duration limits. |
| **Media generation** | Provider-specific asynchronous operations, durable artifact state, progress, cancellation, and terminal continuation. |

One route can serve several workloads, but route admission remains specific to the request.

## Product request envelope

Before calling a provider, Flip constructs a request containing:

- AI identity and applicable instructions;
- selected product context and external evidence;
- admitted tool schemas and trusted runtime scope;
- output and terminal-response requirements;
- model and provider route identifiers;
- modality and artifact references;
- context, time, turn, usage, and cost limits;
- activity and continuation identity;
- tracing and evaluation metadata that does not expose private credentials.

The request is the product contract. A provider adapter should not load additional product data or expand the tool list independently.

## Route selection

Route selection considers:

- required modalities and output types;
- context size and reserved tool or output budget;
- tool-call and structured-output behavior;
- privacy and local-processing requirements;
- provider data-handling policy;
- latency, throughput, and cost targets;
- current endpoint health and queue state;
- route-specific evaluation for the product surface;
- configured fallback order and restrictions.

A route that is healthy but lacks a required modality or tool protocol is not compatible. A cheaper route that changes privacy or evidence requirements is not an automatic fallback.

## Provider adapters

Adapters translate the product request to provider-specific APIs and normalize meaningful events:

- text or structured-output deltas;
- tool calls and tool-result continuation;
- finish and stop reasons;
- provider refusal and safety responses;
- timeout, disconnect, protocol, and malformed-output failures;
- usage and cost data where available;
- asynchronous job identifiers, progress, callbacks, and terminal media results.

Adapters preserve provider differences in capability metadata. Emulated behavior is not reported as identical to native behavior when the distinction affects cancellation, continuation, or correctness.

## Local inference

Local routes can provide privacy, predictable marginal cost, control over model versions, and access when hosted services are unavailable. They also introduce operational limits:

- model process lifecycle and health;
- finite GPU memory and throughput;
- queue and concurrency management;
- model loading and route warm-up;
- context and batch limits;
- local tool and multimodal compatibility;
- host and network failure.

Flip treats local endpoints as routes with explicit health and capability. It does not assume that a configured URL is available or that every local model can execute the same tools and output contracts.

## Hosted inference

Hosted routes provide additional models, modalities, and managed capacity. They require controls for credentials, rate limits, provider outages, data handling, cost, and API behavior changes.

Credentials stay in application configuration and are not included in model-visible context, logs, or reader-facing activity. Provider-specific errors are classified into product-relevant states such as retryable unavailability, deterministic refusal, invalid request, quota, or incompatible output.

## Fallback

Fallback is allowed only when the replacement route preserves the request's required properties.

A fallback check can include:

- same required modality and artifact support;
- equivalent or sufficient tool protocol;
- enough working context for the current state;
- compatible privacy and data handling;
- remaining deadline and budget;
- ability to consume completed tool results and continue the existing activity;
- evaluation status for the product surface.

A route failure can also terminate the AI activity and preserve completed evidence and artifacts. Product chat, forum, and file state remain valid even when the AI reply cannot be completed.

## Context and usage accounting

The runtime estimates or measures prompt, tool, and output usage per route. A working budget reserves space for:

- system and AI-identity instructions;
- selected product context;
- tool schemas;
- accumulated tool results;
- terminal output;
- provider-specific overhead.

Context is compacted or reduced before the provider limit is reached. The system should not depend on a provider truncating arbitrary product context.

Usage records can support operational analysis by route, surface, workload, duration, outcome, and cost. Missing provider usage remains distinguishable from zero usage.

## Route evaluation

Evaluation is performed by workload and route, not by one general model score.

Test sets can measure:

- conversation relevance and product-policy compliance;
- context selection and information leakage;
- tool-call choice, schema validity, and argument quality;
- source use and citation correctness;
- structured curation-plan validity;
- document and data result accuracy;
- media request and continuation behavior;
- latency, throughput, cost, refusal, and failure recovery.

Deterministic product tests verify identity, authorization, effect, persistence, and retry invariants independently of model quality. Model evaluation checks the variable behavior inside those controls.

## Operations

Route operations track endpoint health, recent failures, queue or capacity where available, latency, usage, and evaluation status. Configuration changes are versioned or otherwise attributable so an activity can be related to the route policy that admitted it.

A provider response is stored with product-safe metadata. Credentials, hidden provider headers, private prompt state, and sensitive raw errors are excluded from user-visible projections.

[Previous: Data, realtime, and clients](06-data-realtime-and-clients.md) · [Next: Product and synthetic environment boundary](08-production-and-demo-topology.md)
