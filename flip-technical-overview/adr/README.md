# Architecture decision records

These records summarize decisions that affect Flip's public system architecture. They state the context, current decision, consequences, and conditions for revision.

The records are public technical documentation. They do not link to the private implementation repository and do not expose product data, credentials, provider keys, host configuration, prompt and persona state, or administrative access.

| ADR | Decision |
|---|---|
| [0001](0001-public-architecture-boundary.md) | Maintain a private implementation and a separate public architecture portfolio. |
| [0002](0002-separate-demo-environment.md) | Use a separate synthetic technical environment. |
| [0003](0003-modular-monolith.md) | Keep the main product as a modular Phoenix application. |
| [0004](0004-separate-curation-and-ai-authorship.md) | Separate curation of human content from direct AI authorship. |
| [0005](0005-server-authoritative-capability-plane.md) | Select tools and trusted scope on the server. |
| [0006](0006-split-durable-and-ephemeral-realtime.md) | Separate durable, asynchronous, ephemeral, and local client state. |
| [0007](0007-provider-compatible-inference.md) | Use provider-compatible inference behind product-owned routes. |
| [0008](0008-durable-citations-and-artifacts.md) | Store citations and artifacts as durable application objects. |

The broader decision summary is in [Architecture decisions](../docs/10-architecture-decisions.md).
