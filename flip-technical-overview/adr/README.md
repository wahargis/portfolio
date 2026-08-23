# Flip Architecture Decisions

These records document stable architectural choices that affect product behavior, state, authorization, deployment, or recovery. Production data, credentials, proprietary prompt and persona configuration, security-sensitive thresholds, and private implementation chronology are outside their scope.

| ADR | Decision |
|---|---|
| [0001](0001-public-architecture-boundary.md) | Publish selected architecture and implementation evidence without mirroring the private source repository. |
| [0002](0002-separate-demo-environment.md) | Keep the synthetic technical environment separate from product data and authority. |
| [0003](0003-modular-monolith.md) | Use a modular Phoenix application with PostgreSQL as durable product authority. |
| [0004](0004-separate-curation-and-ai-authorship.md) | Represent conversation curation separately from newly AI-authored participation. |
| [0005](0005-server-authoritative-capability-plane.md) | Construct tools, authorization, and effect authority inside the application. |
| [0006](0006-split-durable-and-ephemeral-realtime.md) | Use durable synchronization and ephemeral Channels for different classes of state. |
| [0007](0007-provider-compatible-inference.md) | Keep model and provider routing behind a product-owned inference contract. |
| [0008](0008-durable-citations-and-artifacts.md) | Persist citations, evidence, and artifacts as typed product objects. |

Each decision states the context, selected design, consequences, and conditions that would justify revision.

[← Flip case study](../README.md)
