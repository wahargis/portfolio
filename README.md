# AI systems and infrastructure portfolio

**Four implemented systems for community knowledge, long-running project control, coding-agent fleet orchestration, and self-hosted AI operations.**

Calling a model is easy. Turning model capability into dependable software requires product authority, durable state, recoverable execution, evidence, resource control, and interfaces that remain coherent when the model, provider, client, or process changes.

This portfolio documents four independently useful systems built around those concerns:

- **[Flip](flip-technical-overview/README.md)** is a community product that connects live chat, durable forum knowledge, and explicitly governed AI participation.
- **[Project Manager](project-manager/README.md)** preserves the evolving reasoning state of long-running research and engineering work and uses it to determine what should happen next.
- **[Baton](baton/README.md)** turns full coding-agent harnesses from several vendors into one persistent, steerable fleet organized around a shared Run.
- **[HomeCloud](homecloud/README.md)** operates local accelerators as supervised inference and agent-execution infrastructure rather than a collection of manually managed model servers.

The projects are adjacent, but they are not modules of an invented master platform. Each owns a different domain, remains useful on its own, and has explicit rather than assumed integration boundaries.

![Portfolio system architecture](assets/portfolio-system.svg)

## At a glance

| System | What a user or operator gets | Engineering problem addressed |
|---|---|---|
| **[Flip](flip-technical-overview/README.md)** | One community experience in which people can talk in real time, preserve useful discussion as forum knowledge, ask an AI participant to research or act, and inspect the authorship and evidence behind the result. | Keeping chat, forum, identity, authorization, provenance, asynchronous work, AI tools, generated artifacts, and web/native client state coherent inside one product. |
| **[Project Manager](project-manager/README.md)** | A durable account of hypotheses, experiments, findings, decisions, constraints, contradictions, active phases, and session state, plus computed guidance about the next actionable work. | Preventing long projects from losing their rationale or continuing from obsolete assumptions when task lists and transcripts no longer represent what the project currently believes. |
| **[Baton](baton/README.md)** | One Run through which an orchestrator or human can plan work, choose exact harness/model/effort routes, launch persistent coding workers, answer questions, steer or stop members, inspect evidence, and harvest results. | Preserving the strengths of vendor-native coding harnesses while adding cross-vendor routing, parallel coordination, durable lifecycle, shared context, and one control surface above them. |
| **[HomeCloud](homecloud/README.md)** | Healthy local model capacity, priority-aware request routing, queued GPU workloads, isolated agent environments, checkpoints, and operational visibility over heterogeneous hardware. | Making local inference and autonomous work recoverable and schedulable when model processes, GPU memory, long-running tools, experiments, and user-facing requests compete and fail independently. |

## Flip: from live conversation to durable community knowledge

Chat is where communities usually do their actual thinking, but valuable explanations and decisions disappear into chronology. Forums preserve knowledge, but asking participants to reconstruct every worthwhile live exchange into a polished thread creates too much friction.

Flip treats the two as one product lifecycle. A discussion can remain immediate in chat and later become durable forum structure while retaining the messages and participants that produced it. The product distinguishes structural curation from new AI-authored content so a model cannot silently rewrite community history or speak through a member’s identity.

The same product boundary governs direct AI participation. Flip derives the invoking actor and community scope, computes the capabilities admitted for that interaction, retrieves authorized internal or external evidence, assigns durable identities to citations and artifacts, and persists the final result under an explicit AI participant. Phoenix, PostgreSQL, Oban, Electric, channels, provider adapters, and native clients support that product contract; they are not the product description by themselves.

**What the case study demonstrates:** a substantial real-time application, relational provenance, server-authoritative AI tools and effects, asynchronous artifact lifecycles, provider-independent model routing, and client convergence across web, desktop, and mobile surfaces.

## Project Manager: preserve how a project changes its mind

A normal project tracker can show that a task is open. It usually cannot explain which experiment created the finding that justified the task, which later evidence contradicted that finding, whether the original decision is still usable, or which dependent phase should now be reconsidered.

Project Manager models that missing control state. Its evidence model records hypotheses, experiments, findings, decisions, principles, constraints, literature, feedback, and typed relationships such as support, contradiction, dependency, supersession, derivation, and testing. Its execution model separately records project phases, dependencies, impact, and status. Session, staleness, search, and review operations make the current project state usable by both people and agents.

The knowledge graph is therefore one plane of the system, not its entire purpose. The larger value is the closed loop between evidence, belief revision, decisions, executable phases, and session continuity. The code can identify dependency-satisfied work, rank actionable phases by impact, surface repeated failed or inconclusive experiments, compute bounded numeric confidence from repeated measurements, and prepare potential contradictions for explicit review.

**What the case study demonstrates:** a Rust domain core, portable SQLite state, causal project memory, dependency-aware planning, truth-maintenance support, and shared CLI, MCP, and web access to the same project model.

## Baton: drive a heterogeneous fleet of full coding harnesses

Modern coding products such as Claude Code, Codex, Kimi, GLM-based harnesses, and other agentic tools are not interchangeable text-completion APIs. Each includes its own file and shell tools, permission model, context management, session behavior, model controls, and vendor-specific strengths. Replacing those harnesses with a lowest-common-denominator worker loop discards much of their value.

Baton keeps the full harness as the worker unit and adds the missing fleet application above it. An orchestrator states an outcome, supplies or approves a plan, pins or constrains harness/model/effort routes, and follows one durable Run while Baton starts isolated workers, carries messages and questions, exposes attention, supports turn-level steering and confirmed stopping, coordinates parallel work, and preserves candidate results.

The coordinator, worktrees, event records, route observations, and verification gates are important because they make this fleet dependable. They are supporting machinery, not a substitute product thesis. A representative Baton run ends with inspectable worker output and coordinator-observed evidence; the system can rerun a declared check in a clean checkout rather than accepting a worker’s claim that the task is complete.

**What the case study demonstrates:** cross-vendor session adapters, run-centric orchestration, exact route control, persistent workers, workflow-as-data, shared control semantics across embedded/CLI/Web/MCP surfaces, lifecycle recovery, and evidence-backed result handling.

## HomeCloud: operate local AI as a runtime

A machine with several GPUs is not automatically an AI platform. Model servers can become unhealthy, requests can contend for the same slots, an experiment can displace an interactive service, generated code can escape the workspace that was meant to contain it, and a long agent can lose its useful state after interruption.

HomeCloud supplies the operating layer around those failures. A supervised instance pool exposes healthy local inference through checkout/checkin semantics. Routing can account for priority, model profile, prompt-cache affinity, throughput, and live slot capacity. GPU claims, workload queues, exclusion locks, and model-service lifecycle controls keep text, media, research, and maintenance workloads from treating hardware ownership as first-come-first-served.

The execution engine owns the environment and lifecycle of autonomous, interactive, and plan-only agent work. It can create a project-derived sandbox, dispatch typed tools, report execution phases, detect unproductive loops, checkpoint progress, resume from durable state, and clean up resources even when a run fails. Research and optimization workloads use this runtime; they do not redefine the platform as a research demo.

**What the case study demonstrates:** Elixir/OTP supervision, local model pooling, heterogeneous GPU scheduling, isolated and recoverable agent execution, durable context and checkpoints, telemetry, and an explicit boundary between operating capability and experimental consumers.

## Engineering themes across the portfolio

### Code owns authority; models supply judgment

Models interpret requests, choose among admitted capabilities, compare evidence, propose plans, and compose results. Code owns identity, authorization, resource admission, durable effect schemas, lifecycle, retry policy, validation, and persistence.

### Durable state is domain-specific

A transcript is not enough. Flip persists authorship, citations, artifacts, and curation lineage. Project Manager persists evidence and decision causality. Baton persists Runs, workers, interactions, results, and recovery state. HomeCloud persists execution, checkpoints, claims, and runtime records.

### Failure is represented, not narrated away

A tool timeout, failed provider request, unhealthy model endpoint, contradicted finding, interrupted worker, or rejected client mutation should change explicit state. The systems avoid presenting a process return value or model assertion as proof that the intended product effect occurred.

### Interfaces project one underlying system

Where a project exposes web, native, CLI, MCP, or embedded access, those surfaces are intended to preserve the same domain objects, lifecycle, authorization, and refusal semantics rather than becoming separate partial implementations.

### Supporting mechanisms remain subordinate

Event logs, worktrees, queues, schedulers, sandboxes, and verification matter because they make a product or runtime capability dependable. Documentation should explain their role without allowing a mechanism inventory to replace the system’s purpose.

## Review paths by audience

| Audience | Recommended path |
|---|---|
| **Recruiter or general interviewer** | Read the four project introductions and representative workflows. They establish the problem, user value, scope, and implemented system shape without requiring framework-specific knowledge. |
| **Software or IT interviewer** | Continue into architecture, state ownership, failure behavior, interfaces, operational constraints, and limitations. The emphasis is on why the boundaries exist and how the assembled system behaves. |
| **AI technical lead** | Focus on model/tool authority, context selection, evidence and provenance, route policy, long-running agent lifecycle, evaluation, recovery, and the distinction between deterministic controls and model judgment. |

The cross-project responsibility map and optional integration contracts are documented in [Portfolio Architecture](PORTFOLIO_ARCHITECTURE.md).

## Source availability

Source availability is intentional and differs by project:

| Project | Source status |
|---|---|
| **Flip** | The product/server implementation remains private. This repository provides the public architecture case study, diagrams, product references, synthetic technical scenarios, and explicit limitations without linking to a private repository. |
| **Project Manager** | The implementation is public in the [Project Manager repository](https://github.com/wahargis/project-manager). |
| **Baton** | Selective public release is planned at [flip-engineering/baton](https://github.com/flip-engineering/baton). That organization repository has not yet been published; this portfolio does not link to the private personal development repository. |
| **HomeCloud** | The implementation and host-specific operations remain private. The public case study describes the stable runtime architecture and deployment constraints without exposing the private repository. |

This repository is a curated architecture portfolio, not a source mirror. It omits credentials, production data, private messages, host-specific secrets, proprietary deployment state, security-sensitive policy, and low-value implementation chronology. The [portfolio notice](NOTICE.md) defines the documentation and licensing boundary.