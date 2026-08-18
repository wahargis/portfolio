# Public architecture decisions

These ADRs record stable, reviewer-relevant decisions. They omit production data, deployment secrets, proprietary prompt/persona state, and internal implementation chronology while linking to the canonical source repository.

| ADR | Decision |
|---|---|
| [0001](0001-public-architecture-boundary.md) | Curate the architecture review path without mirroring the full source tree. |
| [0002](0002-separate-demo-environment.md) | Keep a synthetic technical environment separate from product data and authority. |
| [0003](0003-modular-monolith.md) | Use one modular Phoenix application and PostgreSQL authority. |
| [0004](0004-separate-curation-and-ai-authorship.md) | Separate authorship-preserving curation from explicitly AI-authored participation. |
| [0005](0005-server-authoritative-capability-plane.md) | Compute tools, authorization, and effects server-side. |
| [0006](0006-split-durable-and-ephemeral-realtime.md) | Use durable synchronization and ephemeral channels for different state classes. |
| [0007](0007-provider-compatible-inference.md) | Keep model/provider routing behind a product-owned inference contract. |
| [0008](0008-durable-citations-and-artifacts.md) | Persist evidence and artifacts as product objects rather than prose decorations. |

ADRs explain why a choice was made and what evidence would justify revisiting it. They are not release notes.
