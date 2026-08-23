# ADR 0005: Server-authoritative capability admission

- **Status:** Accepted
- **Scope:** Agent tools, retrieval, and product effects

## Context

Flip contains many product, research, document, data, and media capabilities. A model should not receive every tool or supply trusted actor, community, origin, workspace, and authorization values in tool arguments.

Capabilities also vary by product surface, feature configuration, provider route, current health, and effect policy.

## Decision

The server builds a turn-specific capability catalog using trusted application state. Tool schemas contain user-controlled arguments only where appropriate. The runtime injects actor, community, origin, AI identity, workspace, and other trusted scope.

Each tool has explicit metadata for read or effect behavior, modality, route compatibility, timeout, retry, lifecycle, and result type. Protected reads and effects perform current authorization through the owning product service.

## Consequences

Each product surface and tool class requires admission logic and tests. The model cannot dynamically discover an unrestricted internal API.

The application can prevent scope escalation, remove unavailable capabilities, and apply different controls to retrieval, synchronous effects, and long-running artifacts.

## Revision conditions

A replacement policy system must accept the same trusted inputs, produce inspectable admission and refusal decisions, and avoid moving authorization into model-generated data.
