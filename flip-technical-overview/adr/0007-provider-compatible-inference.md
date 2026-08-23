# ADR 0007: Product-owned inference routes and provider adapters

- **Status:** Accepted
- **Scope:** Local and hosted model execution

## Context

Flip uses different models and providers for conversation, research, structured planning, document work, vision, and media generation. Provider APIs differ in streaming, tool calls, context, structured output, usage, errors, and asynchronous operation.

Product identity, access, tools, effects, and persistence should not change when a route changes.

## Decision

Flip constructs a provider-independent request containing selected context, AI identity and instructions, admitted tools, output requirements, deadlines, route metadata, and activity identity.

Provider adapters translate the request and normalize meaningful events while preserving capability differences. Route selection and fallback consider modality, context, tools, privacy, health, latency, cost, and route-specific evaluation.

## Consequences

Adapters and evaluations require maintenance as providers change. Not every provider can be substituted for every workload. Fallback can fail when no compatible route remains.

The product can use local and hosted inference without giving providers authority over product scope or durable effects.

## Revision conditions

Extend the request or event contract when a provider capability cannot be represented accurately. Do not bypass the product runtime for convenience.
