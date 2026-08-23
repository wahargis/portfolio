# Agent Engineering and AI Systems Portfolio

This portfolio covers four systems I designed and implemented across product-integrated agents, long-running project state, multi-agent software execution, and self-hosted AI infrastructure.

The project pages describe the systems through their execution paths, state models, control logic, failure handling, and implementation boundaries. They begin with the product or operational problem, then show how the software handles model behavior as part of a larger system.

![Execution paths across the four portfolio projects](assets/portfolio-system.svg)

## Projects

| Project | System | Main engineering work |
|---|---|---|
| **[Flip](flip-technical-overview/)** | A real-time community product with AI participants, retrieval, tools, citations, multimodal artifacts, durable background work, and chat-to-forum curation. | Product-integrated agent runtime design; actor-scoped context and retrieval; governed tool and effect execution; evidence and activity records; asynchronous continuation; realtime web and native-client state. |
| **[Project Manager](project-manager/)** | A local project-control system for research and technical work that must remain understandable across many sessions, experiments, decisions, and handoffs. | Typed project and evidence graphs; causal and branch history; contradiction and confidence review; graph-aware retrieval; session continuity; deterministic phase scheduling; shared CLI, MCP, and web behavior. |
| **[Baton](baton/)** | A fleet controller for persistent coding-agent sessions across several coding harnesses. | Goal and plan compilation; route and capability selection; dependency-driven waves and workflows; persistent session control; structured questions, approvals, waits, interruption, and steering; shared context; worktree isolation; result evidence and adoption. |
| **[HomeCloud](homecloud/)** | A supervised local-first AI application runtime operating self-hosted models and agent workloads on finite GPU infrastructure. | OTP supervision; model-instance pools; GPU ownership and workload scheduling; agent execution modes; context and tool infrastructure; containerized workspaces; checkpoint recovery; research, document, connector, and multimodal services. |

## Flip

Flip integrates AI into a stateful multi-user product rather than placing an assistant beside it. An AI turn begins with a product event and an invoking actor. The server derives the applicable account, community, room or forum, membership, feature configuration, and AI identity before context or tools are selected. Internal retrieval remains limited to content the actor can read. Product actions and generated artifacts pass through typed application services and receive durable identities and lifecycle state.

The agent runtime supports direct replies, internal and external research, document and data work, image and video operations, product actions, and continuation after asynchronous provider work. Citations, artifacts, tool activity, and terminal outcomes are stored outside the provider conversation. The same application also manages real-time chat, forums, search, moderation-related controls, background jobs, and client synchronization, so AI behavior remains part of the existing product state and permission model.

**[Read the Flip case study](flip-technical-overview/)**

## Project Manager

Project Manager stores the reasoning and execution state of long technical work. Findings, experiments, hypotheses, decisions, constraints, literature, principles, phases, reviews, and handoffs are represented as typed objects with explicit relationships. This supports queries about what produced a result, what changed a decision, what contradicts a current belief, what was superseded, and what downstream work depends on it.

The system combines that project record with session orientation, graph-aware retrieval, review operations, and a deterministic phase DAG. The evidence model and the execution scheduler are related but remain separate: contradiction or confidence analysis provides review information, while phase readiness follows persisted dependencies and state. A shared Rust and SQLite core exposes the same project semantics through CLI, MCP, and web interfaces.

**[Read the Project Manager case study](project-manager/)**

## Baton

Baton coordinates complete coding-agent sessions rather than reducing agents to one-shot model calls. A run starts from an approved goal and plan. Baton selects an exact harness, model, and effort route, starts persistent workers, schedules dependency-ready work, records events and interactions, and maintains a single run view across CLI, MCP, and web control surfaces.

Workers operate through provider-specific adapters and isolated Git worktrees. Questions, approvals, dependency waits, checkpoints, interruptions, and steering are represented as control state rather than inferred from prose. Shared context and coordination services let work move between sessions without direct worker-to-worker authority. Results remain attached to the approved plan, workspace, verification evidence, review, and explicit adoption or integration decisions.

**[Read the Baton case study](baton/)**

## HomeCloud

HomeCloud turns a four-GPU local AI server into an application runtime for interactive inference, agents, research, documents, connectors, and media work. The OTP supervision tree manages model services, instance pools, GPU monitoring and claims, workload scheduling, tool and modality registries, browser and connector services, agent processes, health checks, and the Phoenix application.

Requests are admitted according to workload type, priority, model capability, healthy instance capacity, and current GPU ownership. Agent work can run in autonomous, interactive, or plan-only modes with selected context, permitted tools, containerized workspaces, loop and quality checks, lifecycle telemetry, and phase-aware checkpoints. Long-running work can resume from stored execution state instead of relying on one process or provider context remaining alive.

**[Read the HomeCloud case study](homecloud/)**

## Engineering coverage

| Area | Portfolio work |
|---|---|
| **Agent runtime design** | Product-event admission, actor and object scope, context construction, retrieval, model routing, tool loops, effects, terminal composition, continuation, and recovery. |
| **Multi-agent orchestration** | Goal and plan contracts, task topology, waves, workflow interpretation, route capability, persistent sessions, interaction state, shared context, attention, and run lifecycle. |
| **Durable reasoning and project state** | Typed evidence, causal history, contradiction and confidence review, provenance, branch and merge history, dependency state, retrieval, and session handoff. |
| **AI infrastructure and operations** | Supervision, model processes, capacity pools, GPU claims, priority queues, workload transitions, sandboxes, checkpoints, health, telemetry, and local or remote routing. |
| **Application and data systems** | Phoenix and OTP applications, Rust services, Node.js control systems, PostgreSQL, SQLite, Oban, realtime channels, synchronization, CLI, MCP, web, desktop, and mobile surfaces. |
| **Reliability and evaluation** | Idempotent effects, explicit lifecycle state, bounded retries, interruption, stale-state rejection, recovery after restart, artifact verification, deterministic tests, and route-specific model evaluation. |

## Source availability

Project Manager and Baton link to their public implementation repositories from their case-study pages. Flip and HomeCloud are private implementations. Their public technical material is contained in this portfolio and does not link to unavailable repository paths.

For a comparison of the four systems' execution and state models, see **[Agent engineering and AI systems design](PORTFOLIO_ARCHITECTURE.md)**.

---

**William Hargis — Agent Engineering, AI Systems, and Data Engineering**
