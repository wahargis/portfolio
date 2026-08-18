# AI systems portfolio

**Product systems, long-horizon project memory, cross-harness agent orchestration, and self-hosted AI infrastructure.**

This portfolio documents four independently useful systems built at different layers of AI-assisted work:

- **Flip** is a real-time community product that connects chat, forums, curation, AI participation, retrieval, provenance, and generated artifacts.
- **Project Manager** gives long-running research and engineering work a typed evidence graph, causal decisions, contradiction handling, and computed next actions.
- **Baton** is a cross-harness fleet driver: one orchestrator directs persistent full-session coding harnesses across vendors with planning, routing, communication, steering, shared context, workflow composition, and reusable fleet capabilities.
- **HomeCloud** is a self-hosted AI runtime that supplies supervised local inference, GPU scheduling, sandboxed agents, memory, recovery, and research infrastructure.

They are not components of one invented platform. Each system has its own product boundary and remains useful on its own; their integration points are narrow and explicit.

![Portfolio system architecture](assets/portfolio-system.svg)

## Four systems, four different problems

| System | Primary problem | Architectural center |
|---|---|---|
| **[Flip](flip-technical-overview/README.md)** | Live conversation is valuable but transient, while AI participation inside a social product introduces authorship, permission, evidence, and lifecycle problems. | Phoenix/PostgreSQL product domains, real-time sync, background curation, governed AI participation, retrieval, citations, artifacts, and web/native clients. |
| **[Project Manager](project-manager/README.md)** | Long-running work loses hypotheses, evidence, contradictions, decision rationale, and session continuity when reduced to task lists or transcripts. | Rust domain core, typed evidence graph, dependency DAG, truth maintenance, SQLite, and shared CLI/MCP/web surfaces. |
| **[Baton](baton/README.md)** | Full coding harnesses are powerful execution environments, but they are isolated vendor products with incompatible control surfaces and no shared fleet application. | Run-centric orchestration, exact harness/model/effort routing, persistent workers, waves and workflows, messaging and steering, shared context/memory, agent-shaped capabilities, and a reliable fleet kernel. |
| **[HomeCloud](homecloud/README.md)** | Local models and autonomous workloads need supervision, fair GPU access, isolation, context continuity, and recoverable execution. | Elixir/OTP supervision, local model pools, slot-aware inference routing, GPU scheduling, sandboxed agents, RAG/memory, checkpoints, and evaluation workloads. |

## What the portfolio demonstrates

### Systems around models, not model demos

Each project treats a model as one participant in a larger system:

- Flip embeds models inside a social product with durable authorship and permission semantics.
- Project Manager stores project knowledge as typed, revisable state rather than relying on an agent’s transcript.
- Baton preserves the strengths of vendor-native coding harnesses and composes them into a persistent heterogeneous fleet.
- HomeCloud turns local inference and agent execution into supervised infrastructure.

### Durable state carries more than conversation

The systems persist the state their domain actually needs:

- Flip retains source-message links, forum provenance, AI artifacts, citations, and product state.
- Project Manager persists experiments, findings, hypotheses, decisions, constraints, literature, sessions, and typed relationships.
- Baton retains Goals, Plans, Runs, worker sessions, attention state, task/workflow knowledge, shared coordination state, artifacts, evidence, and recovery information.
- HomeCloud persists tasks, checkpoints, documents, memories, learned facts, skills, knowledge-graph relationships, and research results.

### Interfaces preserve domain semantics

CLI, MCP, web, native, and embedded surfaces are projections of one underlying system wherever a project exposes more than one interface. New surfaces are not allowed to invent weaker lifecycle, permission, or state semantics.

### Supporting machinery stays subordinate to the product

Verification, event logs, worktrees, queues, schedulers, and deployment isolation matter because they make a product capability dependable. They are not substituted for the product’s actual aim. This is especially important for Baton: the fleet driver is the product; the coordinator and verification layers support the driving.

## System relationships

The useful composition points are intentionally narrow:

- Flip can route inference to hosted providers or a HomeCloud-hosted compatible endpoint.
- Baton can direct local or remote coding harnesses and can consume HomeCloud-provided model or sandbox capacity without depending on HomeCloud.
- Project Manager can receive selected findings, decisions, constraints, and handoffs from Baton runs or HomeCloud experiments without becoming either system’s runtime.
- HomeCloud can serve unrelated applications without knowing their product or project semantics.

See [Portfolio Architecture](PORTFOLIO_ARCHITECTURE.md) for the responsibility map and integration contracts. The [portfolio notice](NOTICE.md) defines the documentation and licensing boundary.

## Review paths

**Three minutes:** read this page and the first screen of each case study.

**Fifteen minutes:** follow the architecture and representative workflow in each case study.

**Deep review:** use Flip’s technical chapters, Project Manager’s knowledge model, Baton’s full-system source and fleet architecture, and HomeCloud’s platform/research boundary.

## Canonical source repositories

- [Flip](https://github.com/wahargis/flip)
- [Project Manager](https://github.com/wahargis/project-manager)
- [Baton](https://github.com/wahargis/baton)
- [HomeCloud](https://github.com/wahargis/home-cloud)

This repository is a curated architecture portfolio, not a substitute for the source repositories. It omits credentials, production data, host-specific secrets, proprietary deployment state, and low-value implementation chronology while linking to canonical public source where available.
