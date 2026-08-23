# Agent Engineering and AI Systems Portfolio

Four implemented systems spanning AI product architecture, agent execution and orchestration, durable research state, data modeling, and local inference operations. Each project page follows the product or operating problem into its state model, execution model, failure handling, and representative implementation evidence.

## Projects

| Project | System | Main engineering work |
|---|---|---|
| **[Flip](flip-technical-overview/)** | A real-time community product with AI participants, research tools, multimodal artifacts, and chat-to-forum curation. | Product-integrated agent execution, actor-scoped context and retrieval, typed tools and effects, asynchronous continuation, provenance, authorization, and web/native client convergence. |
| **[Project Manager](project-manager/)** | A local project-intelligence system for research and technical work that continues across many sessions. | Typed project and evidence graphs, causal and branch history, contradiction and confidence review, graph-aware retrieval, session continuity, deterministic phase scheduling, and shared CLI/MCP/web behavior. |
| **[Baton](baton/)** | A run-centric control system for persistent coding-agent sessions across several coding harnesses. | Goal and plan compilation, route and capability selection, dependency-driven waves and workflows, structured interaction, shared context, worktree ownership, result harvesting, review, adoption, and integration. |
| **[HomeCloud](homecloud/)** | A supervised local-first AI application platform operating self-hosted models and agent workloads on finite GPU infrastructure. | OTP supervision, model-instance pools, GPU claims and scheduling, agent execution modes, tool and modality registries, containerized workspaces, checkpoint recovery, research, documents, connectors, and media services. |

## Flip

Flip is a multi-user community application in which AI participates through the same product identity, authorization, data, and lifecycle systems used by human participants. A turn begins with a product event and an invoking actor. The server resolves the AI identity, community, room or forum, membership, visibility, feature state, and permitted capabilities before selecting context or calling a model.

The runtime supports direct replies, internal and external research, document and data work, product actions, image and video operations, generated artifacts, and continuation after asynchronous provider work. Tool calls are typed application requests. Product mutations, citations, and artifacts receive durable identities and are committed through the owning domain services.

Conversation curation is a separate workflow. Selected human messages can be organized into forum content while retaining participant attribution, source-message relationships, source-derived access restrictions, feedback, and recuration history.

**[Flip case study](flip-technical-overview/)** · **[Conversation-to-forum lifecycle](flip-technical-overview/diagrams/conversation-to-forum.svg)**

## Project Manager

Project Manager stores technical project state as typed objects and relationships rather than relying on notes, issue comments, or prompt history. Experiments, findings, hypotheses, decisions, constraints, literature, principles, phases, reviews, branches, merges, and handoffs remain queryable across sessions.

The evidence graph records support, contradiction, supersession, derivation, production, testing, dependency, causal, branch, merge, and provenance relationships. Retrieval combines lexical matching with typed filters and graph traversal. Session start and end operations load current state and write durable handoffs for the next human or agent session.

The execution DAG remains separate from the evidence graph. Completed dependencies determine which phases are ready, while contradiction and confidence analysis provide review information that can lead to explicit project updates.

**[Project Manager case study](project-manager/)** · **[Typed project record](project-manager/assets/project-record.svg)** · **[Public source](https://github.com/wahargis/project-manager)**

## Baton

Baton coordinates persistent coding-agent sessions instead of treating workers as one-shot model calls. A run starts from an approved goal and plan with repository scope, dependencies, route policy, budgets, effects, and verification requirements.

The controller selects exact harness, model, and effort routes; schedules dependency-ready work; starts workers in owned Git worktrees; and records questions, approvals, waits, checkpoints, interruptions, steering, events, shared context, and result lineage. Provider adapters preserve differences in native session behavior rather than hiding them behind a text-completion interface.

Worker completion and workflow completion remain separate. Results are content-addressed and harvested against declared contracts; verification, review, adoption, integration, export, and cleanup are explicit later operations.

**[Baton case study](baton/)** · **[Run lifecycle](baton/assets/run-lifecycle.svg)** · **[Public source](https://github.com/Flip-Engineering/baton)**

## HomeCloud

HomeCloud operates a four-GPU V100 server as a shared application runtime for interactive inference, agent tasks, research, document work, connectors, browser and search tools, and multimodal workloads.

The OTP application supervises model services, instance pools, GPU monitoring and claims, workload scheduling, agent processes, tool and modality registries, sandboxes, checkpoints, research services, connectors, media queues, health monitoring, and Phoenix application surfaces.

Requests are admitted according to workload type, priority, route compatibility, healthy instance capacity, queue state, and GPU ownership. Agent work can run in autonomous, interactive, or plan-only modes with selected context, permitted tools, containerized workspaces, loop controls, lifecycle events, and phase-aware checkpoints.

**[HomeCloud case study](homecloud/)**

## Engineering coverage

| Area | Portfolio work |
|---|---|
| **Agent runtime design** | Product-event admission, actor and object scope, context construction, retrieval, model routing, tool loops, effects, terminal validation, continuation, and recovery. |
| **Multi-agent orchestration** | Goal and plan contracts, task topology, waves, workflows, route capability, persistent sessions, questions, approvals, waits, interruption, steering, shared context, harvesting, and run lifecycle. |
| **Durable project reasoning** | Typed evidence, causal and branch history, contradiction and confidence review, provenance, graph-aware retrieval, temporal state, phase dependencies, and session handoff. |
| **AI infrastructure** | Supervision, model processes, capacity pools, priority queues, GPU ownership, workload transitions, sandboxes, checkpoints, health, telemetry, and local or remote routing. |
| **Application and data systems** | Phoenix and OTP applications, Rust services, Node.js control systems, PostgreSQL, SQLite, Oban, realtime channels, synchronization, CLI, MCP, web, desktop, and mobile surfaces. |
| **Reliability and evaluation** | Idempotent effects, explicit lifecycle state, bounded retry, stale-state rejection, restart recovery, artifact verification, deterministic tests, route-specific model evaluation, and operational telemetry. |

## Representative system evidence

- Flip documents a direct AI turn and a separate conversation-to-forum lifecycle with source relationships, operation state, and recuration.
- Project Manager includes the public `atlas` quickstart plus a broader typed project record connecting phases, experiments, findings, hypotheses, decisions, constraints, sessions, and review.
- Baton includes a recorded two-member wave in which both sessions reached `work_completed`, one required result was missing from the harvest, and the workflow correctly ended incomplete; the lifecycle diagram shows how a normal run proceeds through capture, verification, review, and integration.
- HomeCloud includes the four-V100 deployment, separate A100 Drive accelerator, model-instance capacity, GPU scheduling, agent execution, checkpointing, and application-service flow.

## System boundaries

- The four projects are independent systems. None requires the others to operate.
- Flip and HomeCloud are private implementations. Their public technical material is contained in this portfolio and does not link to unavailable source repositories.
- Project Manager and Baton link to their public implementation repositories from their project pages.
- Public material omits product data, credentials, provider keys, private messages, host-specific secrets, prompt and persona configuration, and administrative authority.

The systems’ responsibility boundaries and possible standards-based integration surfaces are summarized in **[System boundaries and interfaces](PORTFOLIO_ARCHITECTURE.md)**.

## Live systems

- [Flip product](https://flip.engineering)
- [Synthetic Flip technical environment](https://flip.tech-demo.dev)
