# ADR 0007 — Provider-compatible inference boundary

- **Status:** Accepted
- **Decision scope:** Model routing

## Context

Flip uses models for several surfaces with different latency, context, tool, and modality requirements. Provider APIs and tool-call behavior change over time. Product semantics should not be tied to one model vendor.

## Decision

Define a product-owned inference envelope and normalize provider behavior behind adapters. Route policy selects a compatible local or hosted endpoint based on surface requirements and deployment policy.

Identity, permissions, admitted tools, citations, terminal behavior, and persistence remain in Flip.

## Consequences

Adapters must handle tool-call, streaming, finish-reason, forced-tool, and error-shape differences. Models can be evaluated and replaced by surface without redesigning the product.

## Revisit when

A stable cross-provider protocol removes meaningful adapter differences while preserving current product control.
