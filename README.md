# AI systems portfolio

**Product, knowledge, fleet-orchestration, and compute systems for long-running AI-assisted work.**

This portfolio documents four independently useful systems built around different problems:

- **Flip** turns live community conversation into durable knowledge while supporting governed AI participation inside the product.
- **Project Manager** preserves the evolving evidence, beliefs, decisions, and next actions of long-running research and engineering programs.
- **Baton** lets one full coding harness direct a heterogeneous fleet of other full-session harnesses, with live messaging, telemetry, mid-run steering, shared coordination state, and durable result handoff.
- **HomeCloud** turns local accelerators into supervised inference and agent infrastructure with scheduling, isolation, context continuity, and recovery.

The systems can work together, but they are not presented as one invented master platform. Each owns a distinct architectural problem and exposes narrow integration boundaries.

![Portfolio system architecture](assets/portfolio-system.svg)

## Four systems, four responsibilities

| Plane | System | Primary problem | Architectural center |
|---|---|---|---|
| **Product** | [Flip](flip-technical-overview/README.md) | Live conversation is valuable but transient, while AI participation introduces authorship, permission, evidence, and durability problems that a chatbot endpoint does not solve. | One product authority for chat, forum, curation, AI participants, provenance, artifacts, and synchronized clients. |
| **Knowledge** | [Project Manager](project-manager/README.md) | Long projects lose hypotheses, contradictory evidence, decision rationale, and continuity when represented as task lists or transcripts. | A typed evidence graph and dependency model that computes what the project believes and what should happen next. |
| **Fleet orchestration** | [Baton](baton/README.md) | Vendor coding harnesses are powerful but isolated; one harness cannot ordinarily direct several others as persistent, observable, steerable workers. | A run/wave/workflow application over full-harness adapters, live communication and steering, shared coordination state, and reliable lifecycle plumbing. |
| **Compute / runtime** | [HomeCloud](homecloud/README.md) | Local models and autonomous workloads need supervised endpoints, fair GPU access, isolation, recoverable execution, and durable context. | Elixir/OTP supervision around inference slots, workload scheduling, agent sandboxes, memory, checkpoints, and research workloads. |

## What the work demonstrates

### Model reasoning is useful without being made system authority

The projects put models in roles where judgment is valuable while keeping the surrounding system responsible for durable semantics:

- Flip owns identity, permissions, authorship, citations, and product effects.
- Project Manager owns the project’s evidence relationships and belief state.
- Baton leaves planning and intervention to the orchestrator harness while a non-model coordinator makes fleet commands, lifecycle, and shared state dependable.
- HomeCloud owns capacity, endpoint health, sandbox lifecycle, and recovery around local inference.

### Durable state preserves meaning, not just output

The systems retain the relationships needed to understand and continue work:

- Flip links durable forum knowledge, AI replies, citations, and artifacts to their sources and authors.
- Project Manager links experiments, findings, hypotheses, constraints, and decisions through explicit evidence relations.
- Baton retains run/wave identity, worker interaction, progress, shared notes, result artifacts, recovery state, and orchestration decisions across full harness sessions.
- HomeCloud preserves task state, checkpoints, retrieved documents, memory, research trials, and infrastructure state across process boundaries.

### Interfaces project one domain model

Web, native, CLI, MCP, embedded, and automation surfaces are not treated as separate products with divergent semantics. Where several surfaces exist, they project the same underlying authority and lifecycle.

### Evidence is designed into the workflow

The portfolio favors evidence that survives beyond a model response: source-message relationships, exact citations, typed findings, durable event records, git artifacts, fresh-environment checks, compilation, deterministic tests, and hardware-backed measurements.

## System relationships

The useful connections are narrow and asymmetric:

- Flip can route inference to a hosted endpoint or a HomeCloud-hosted compatible endpoint without giving the inference layer product authority.
- Baton can drive local or remote full-session harnesses, including workers backed by HomeCloud capacity, without reducing those harnesses to raw model calls.
- Selected Baton run results can become Project Manager findings, decisions, or constraints; raw fleet telemetry does not automatically become project truth.
- Project Manager can preserve the rationale around work performed in Flip, Baton, or HomeCloud without owning their runtime behavior.

See [Portfolio Architecture](PORTFOLIO_ARCHITECTURE.md) for responsibility boundaries and composition contracts. The [portfolio notice](NOTICE.md) defines the publication and licensing boundary.

## Review paths

**Three minutes:** read this page and the opening sections of each case study.

**Fifteen minutes:** follow the principal architecture and one representative lifecycle in each project.

**Deep review:** use the Flip technical chapters, Project Manager evidence model, Baton fleet-driver architecture, and HomeCloud platform/research boundary.

## Publication boundary

This repository publishes product semantics, architecture, sanitized workflows, engineering decisions, diagrams, maturity boundaries, and limitations. It does not publish production data, credentials, proprietary prompts, host-specific configuration, or private implementation merely to make the documentation look exhaustive.

Project availability differs:

- Project Manager and Baton have public source repositories linked from their case studies.
- Flip and the broad HomeCloud implementation are documented here without mirroring private commercial or host-specific source.
- Supporting clients, adapters, and tools are explained inside the flagship system they serve rather than inflated into additional flagship projects.
