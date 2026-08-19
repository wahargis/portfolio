# HomeCloud

> A local-first AI application and operations platform that turns finite GPU hardware into dependable inference, agent, research, and multimodal services.

## At a glance

| | |
|---|---|
| **Product** | A supervised runtime and application platform for self-hosted models, agent execution, research workflows, documents, connectors, and media services. |
| **Users** | A technical operator and the applications or agents that need reliable access to local and remote AI capability. |
| **Core problem** | Local accelerators provide valuable compute but not an operating model. Applications still need model lifecycle, capacity allocation, priority, health, recovery, isolation, and a coherent API over heterogeneous backends. |
| **Engineering focus** | GPU claims, model-instance pools, priority-aware routing, workload scheduling, sandboxed tools, recoverable agent loops, verification, and supervised application services. |
| **Primary implementation** | Elixir/OTP and Phoenix with PostgreSQL/Ash, local and remote inference adapters, system process integration, containers, telemetry, and LiveView application surfaces. |
| **Source** | [`wahargis/home-cloud`](https://github.com/wahargis/home-cloud) |

## The product problem

Running a model locally is straightforward compared with operating a useful local AI platform.

A single inference server does not answer:

- Which model service should be running on which accelerator?
- How many concurrent requests can the healthy hardware actually support?
- Should an interactive user request preempt a research benchmark or background job?
- How does an application check out a model instance and return it safely?
- What happens when a server process is unhealthy or a GPU is already claimed?
- How are long-running agents isolated, observed, checkpointed, and resumed?
- How do document, connector, research, and media workloads share the same infrastructure?
- When should the system use a local model, a different local backend, or a remote provider?

HomeCloud treats those as one runtime problem. It combines infrastructure control with application-level AI services so that local compute becomes a dependable capability rather than a collection of manually started model processes.

The platform is local-first, not local-only. Its abstractions allow local model servers and remote providers to participate in the same application while preserving the operational distinctions that matter: capacity, latency, cost, health, privacy, and tool support.

## Representative workflow

A representative agent request proceeds as follows:

1. **A request enters through an application surface.** It may originate from the web application, an internal workflow, a connector, or a research process.
2. **The runtime selects an inference path.** Model and inference routers consider configuration, backend type, workload priority, and current capacity.
3. **A healthy instance is checked out.** The instance pool tracks local model-server slots, affinities, load, health, and restart state. Concurrency is derived from healthy infrastructure rather than supplied as an arbitrary caller constant.
4. **GPU authority is respected.** Claim and scheduling services prevent incompatible workloads from treating the same accelerator as unowned. Interactive, research, and background priorities shape admission and model-service changes.
5. **The execution environment is prepared.** Agent work receives a containerized sandbox and a durable workspace. The execution engine builds context, discovers permitted tools, dispatches model turns, executes tools, and detects loops or quality regressions.
6. **State is checkpointed.** Message history, plan state, loop state, and engine state can be persisted by phase so a long-running task can resume after interruption instead of restarting from turn zero.
7. **Results return to product state.** The outcome can be validated, stored, presented in the UI, used by a research workflow, placed in the document vault, delivered through a connector, or handed to a media service.
8. **Capacity is released or returned to baseline.** Checked-out instances and GPU claims are reconciled so later interactive and background work sees accurate availability.

This workflow connects hardware reality to application behavior. The user asks for an AI capability; the platform handles the model, process, resource, tool, recovery, and persistence concerns required to deliver it.

## Architecture

<img src="assets/diagrams/architecture.svg" alt="HomeCloud local-first AI platform architecture" width="1050" />

### 1. Supervised application platform

HomeCloud runs as an OTP application with a broad supervision tree. Database access, PubSub, HTTP clients, model instances, GPU monitors, schedulers, connectors, browser services, model installation, agent execution, research processes, health monitoring, and the Phoenix endpoint are started and supervised according to configuration.

This matters because the platform is not a script that assumes every dependency is already available. Components have lifecycle, restart behavior, optional configuration, and a place in the application’s operational state.

### 2. Model and instance control

The instance pool manages local inference-server processes as check-out resources. It tracks available and busy slots, request priority, model affinity, health, and restart state. User, research, and background work can be treated differently without requiring each caller to implement its own queue.

The routing layer can choose between local server types and remote providers while presenting a coherent inference interface to higher-level workflows. Backend-specific behavior remains in adapters rather than leaking into every application feature.

### 3. GPU claims and workload scheduling

A GPU is not available merely because a process has not recently logged activity. HomeCloud represents claims and workload ownership explicitly, polls hardware and queue state, and applies cooldown and idle checks before changing a model service.

The scheduler can preserve an interactive baseline, move to a requested workload when the hardware is safely idle, and return capacity after work drains. Priority and ownership therefore shape model lifecycle instead of allowing each script to start or stop heavyweight services independently.

### 4. Recoverable agent execution

The execution engine supports autonomous, interactive, and plan-only modes. It builds context, discovers tools by profile, dispatches inference, executes tool calls, streams phase and result events, and monitors repeated actions, repeated output, score regression, and stalled improvement.

Agent tasks run in containerized workspaces with relevant toolchains. The engine owns cleanup and records lifecycle telemetry so a setup failure or helper-process crash does not become an unexplained disappearance.

Checkpointing persists turn history and execution state in PostgreSQL. Phase-specific snapshots allow plan, implementation, and critique work to coexist and allow the freshest valid state to be resumed after a crash.

### 5. Research, verification, and evaluation

The same runtime supports research and optimization work that would otherwise compete informally with interactive use. Research jobs can operate at a lower priority, use shared inference routing, and feed results into persisted application state.

The platform includes tooling for evaluation, verification, optimization, code and CUDA research, and autonomous programs. These are presented as workloads over the runtime—not as unrelated scripts—so they inherit model routing, resource management, telemetry, and recovery behavior.

### 6. Product services above the runtime

HomeCloud extends beyond infrastructure management. The source tree includes document-vault behavior, connectors, browser and search tools, cinema/media workflows, OCR, model installation, notifications, and multimodal adapters.

Those features are not listed to inflate scope. They demonstrate why the runtime exists: multiple product capabilities can reuse the same supervised model, GPU, agent, tool, and persistence layer instead of each shipping its own fragile AI stack.

## Key design decisions

### Derive concurrency from healthy infrastructure

Callers do not decide that three jobs may run merely because they request three. Batch concurrency is derived from available inference capacity. This aligns application parallelism with the model servers and accelerator slots that can actually serve it.

### Give user work explicit priority

The instance pool and scheduler distinguish user, research, and background work. Priority is part of the runtime contract, which allows exploratory or maintenance workloads to use spare capacity without making interactive behavior unpredictable.

### Represent GPU ownership

Claims and scheduling state prevent multiple subsystems from assuming the same accelerator is free. Model-service transitions occur through a controller that can inspect idleness, queues, locks, and cooldown state.

### Isolate agent tools

Agent execution receives a container and workspace rather than unrestricted host execution. File operations are constrained to the mounted workspace, and resource reservations let tool work yield under pressure while still using idle host capacity.

### Checkpoint semantic state, not only logs

A recoverable agent needs more than its printed transcript. Checkpoints include message history, plan and context variables, loop-detection state, and engine state. Resumption can continue the execution model rather than replaying the entire task from the beginning.

### Keep local and remote inference behind explicit adapters

A local llama.cpp server, another local backend, and a remote provider have different operational properties. Adapters give higher-level features a stable interface without erasing backend-specific health and capacity.

### Supervise optional capability

Many HomeCloud services depend on deployment configuration. The application starts optional components deliberately and exposes health and telemetry rather than assuming that every connector, model service, or accelerator is present.

## Reliability and failure handling

The implementation addresses failure at several layers:

- **Unhealthy model instance:** pool health and restart state prevent a failed server from remaining an apparently available slot.
- **GPU contention:** claim and scheduler state gate model-service changes and concurrent workloads.
- **Agent loop:** repeated actions, repeated output, quality regression, and plateaus can trigger warnings or stopping behavior.
- **Tool or setup failure:** execution lifecycle events and status updates preserve diagnostic state; cleanup is designed to run on exceptional paths.
- **Process interruption:** turn-level, phase-aware checkpoints preserve enough state for resumption.
- **Sandbox leakage:** workspaces are mounted into containers, file operations are validated against the workspace, and generated workspaces can use durable, quota-controlled storage.
- **Deployment variance:** optional components and inference backends are configuration-driven rather than assumed.
- **Background starvation of users:** priority queues and baseline-return behavior keep interactive work visible to the scheduler.

## Implementation evidence

| Source | What it demonstrates |
|---|---|
| [`lib/home_cloud/application.ex`](https://github.com/wahargis/home-cloud/blob/main/lib/home_cloud/application.ex) | The supervised application topology spanning database, model services, GPU control, agents, research, connectors, browser/media services, health, and the Phoenix endpoint. |
| [`lib/home_cloud/intelligence/instance_pool.ex`](https://github.com/wahargis/home-cloud/blob/main/lib/home_cloud/intelligence/instance_pool.ex) | Model-instance checkout and return, priority queues, affinity, health, restart behavior, and multi-slot capacity. |
| [`lib/home_cloud/infrastructure/gpu_workload_scheduler.ex`](https://github.com/wahargis/home-cloud/blob/main/lib/home_cloud/infrastructure/gpu_workload_scheduler.ex) | GPU-aware workload selection, idle and cooldown checks, controlled model-service transitions, locks, and return to baseline. |
| [`lib/home_cloud/intelligence/execution_engine.ex`](https://github.com/wahargis/home-cloud/blob/main/lib/home_cloud/intelligence/execution_engine.ex) | Autonomous, interactive, and plan-only agent modes; infrastructure-derived batch concurrency; sandbox ownership; context, tools, loop safety, and lifecycle events. |
| [`lib/home_cloud/intelligence/execution_engine/checkpointer.ex`](https://github.com/wahargis/home-cloud/blob/main/lib/home_cloud/intelligence/execution_engine/checkpointer.ex) | Phase-aware persistence of message, plan, loop, event, and engine state for crash recovery. |
| [`lib/home_cloud/intelligence/agent_sandbox.ex`](https://github.com/wahargis/home-cloud/blob/main/lib/home_cloud/intelligence/agent_sandbox.ex) | Containerized agent workspaces, durable storage, soft resource controls, tool execution, and workspace path validation. |

## What the project demonstrates

For an infrastructure or general software reviewer, HomeCloud demonstrates OTP supervision, process lifecycle, resource ownership, scheduling, health, adapters, persistence, containers, telemetry, and application integration around heterogeneous hardware.

For an AI technical lead, it demonstrates the operational substrate that model-centric demos usually omit: inference capacity, priority, tool isolation, loop safety, checkpoint recovery, evaluation workloads, and the ability to reuse those services across multiple product domains.

For a recruiter or product reviewer, the value can be stated simply: HomeCloud turns owned AI hardware into a usable platform rather than requiring every feature or user to understand how to start, route, monitor, and recover model processes manually.

## Scope and boundaries

- HomeCloud is a self-hosted application platform, not a general-purpose replacement for Kubernetes or a public multi-tenant cloud scheduler.
- Hardware- and model-specific configuration exists in the implementation, but this portfolio page intentionally describes the reusable control model rather than private deployment details.
- Not every optional service is required in every deployment.
- Local-first does not mean every workload must run locally; remote providers remain valid routes where their capabilities or operating characteristics are appropriate.
- This page emphasizes the platform architecture. It does not inventory every agent, research module, connector, media workflow, or UI feature present in the repository.

## Review paths

**Five minutes:** read **The product problem**, **Representative workflow**, and **Key design decisions**.

**Twenty minutes:** continue through **Architecture**, **Reliability and failure handling**, and the six source links.

**Deep review:** trace one request from application startup and routing through instance checkout, GPU scheduling, sandboxed execution, checkpointing, and result persistence.

[← Back to portfolio](../README.md) · [View source repository](https://github.com/wahargis/home-cloud)
