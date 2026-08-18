# AI systems portfolio

**Product, knowledge, control, and compute systems for long-running AI-assisted work.**

This portfolio documents four independently useful systems that address different failure modes in applied AI engineering:

- **Flip** turns live community conversation into durable knowledge while supporting bounded, tool-using AI participants.
- **Project Manager** gives long-running research and engineering programs a typed evidence graph, causal decision record, and computed next actions.
- **Baton** places a deterministic control plane around probabilistic coding agents, including isolation, steering, recovery, and independent verification.
- **HomeCloud** supplies self-hosted inference, GPU-aware scheduling, sandboxed execution, memory, and research infrastructure on local hardware.

They are not presented as one monolith. They are a coherent set of architectural layers with explicit ownership boundaries and composable interfaces.

![Portfolio system architecture](assets/portfolio-system.svg)

## Four systems, four responsibilities

| Plane | System | Primary problem | Architectural center |
|---|---|---|---|
| **Product** | [Flip](flip-technical-overview/README.md) | Live conversation is valuable but transient; generic assistants do not become accountable product participants merely by calling an LLM. | Phoenix/PostgreSQL product domains, real-time sync, background synthesis, governed AI reply runtime, retrieval, tools, provenance, and web/native clients. |
| **Knowledge** | [Project Manager](project-manager/README.md) | Long-running work loses hypotheses, evidence, contradictions, decision rationale, and session continuity. | Rust domain core, typed evidence graph, dependency DAG, truth maintenance, SQLite, and shared CLI/MCP/web surfaces. |
| **Control** | [Baton](baton/README.md) | Full coding agents are capable but probabilistic; subprocess wrappers do not provide lifecycle authority, reliable steering, or trustworthy adoption. | Deterministic coordinator, durable event truth, declarative workflows, exact routes/scopes, isolated worktrees, unified surfaces, and fresh-worktree verification. |
| **Compute** | [HomeCloud](homecloud/README.md) | Local models and autonomous workloads need supervision, fair GPU access, isolation, context continuity, and recoverable execution. | Elixir/OTP supervision, local model pools, slot-aware inference routing, GPU scheduling, sandboxed agents, RAG/memory, checkpoints, and evaluation workloads. |

## What distinguishes the portfolio

### AI reasoning is not system authority

Models propose, interpret, synthesize, and decide within bounded product roles. Plain code owns permissions, state transitions, concurrency, deadlines, process lifecycle, evidence retention, and irreversible effects.

That division appears differently in each system:

- Flip exposes only the tools permitted by the user, community, surface, and deployment configuration.
- Project Manager stores beliefs as typed, revisable evidence rather than treating generated prose as settled truth.
- Baton re-derives worker results in a fresh environment before allowing adoption or integration.
- HomeCloud derives concurrency from live inference capacity and runs tool-using agents in owned sandboxes.

### Durable state carries meaning

Each system persists more than messages or task status:

- Flip retains source-message links, forum provenance, AI artifacts, citations, and product state.
- Project Manager persists experiments, findings, hypotheses, decisions, constraints, literature, sessions, and typed relationships.
- Baton records commands, lifecycle events, interaction state, result pins, verification evidence, and resource ownership.
- HomeCloud persists task state, checkpoints, documents, memory, learned facts, skills, knowledge-graph relationships, and research results.

### Multiple interfaces share one domain model

CLI, MCP, web, native, and automation surfaces are not separate products with divergent semantics. They project the same underlying authority wherever the implementation supports more than one interface.

### Verification precedes trust

The portfolio favors evidence that can be independently inspected or re-derived: database constraints, typed state machines, bounded tool envelopes, immutable source links, fresh-worktree tests, compilation, deterministic test execution, and hardware-backed profiling.

## System relationships

The systems can be deployed independently. Their useful composition points are intentionally narrow:

- Flip can route inference to a remote provider or a HomeCloud-hosted OpenAI-compatible endpoint.
- Baton can dispatch coding or research work to local or remote full-session harnesses, including infrastructure backed by HomeCloud.
- Project Manager can receive durable findings, decisions, constraints, and handoffs from human or agent workflows.
- HomeCloud can host models and execution sandboxes without owning product semantics, research truth, or source-control adoption policy.

See [Portfolio Architecture](PORTFOLIO_ARCHITECTURE.md) for the responsibility map, integration contracts, and deliberate non-goals. The [portfolio notice](NOTICE.md) defines the publication and licensing boundary.

## Review paths

**Three minutes:** read this page and the first screen of each case study.

**Fifteen minutes:** follow the architecture, execution lifecycle, and status sections in the four case studies.

**Deep review:** use the Flip technical chapters, the Project Manager knowledge model, the Baton trust model, and the HomeCloud platform/research boundary.

## Publication boundary

This repository is a reviewer-facing architecture portfolio. It publishes product semantics, system boundaries, sanitized workflows, implementation-informed diagrams, engineering decisions, and honest maturity statements. It does not mirror private commercial source, credentials, private data, model prompts, host-specific configuration, or internal operational thresholds.

Project-specific availability and licensing differ:

- Project Manager has a public source repository linked from its case study.
- Flip, Baton, and the broad HomeCloud implementation are documented here without publishing their private implementation.
- Supporting clients and adapters are folded into the relevant flagship narrative rather than presented as additional flagship systems.
