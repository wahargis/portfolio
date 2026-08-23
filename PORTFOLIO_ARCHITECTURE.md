# Systems Architecture Across the Portfolio

The four projects occupy different layers of AI systems engineering. They are independently deployable products, not modules of one required platform.

| Project | Product or operating boundary | Agent-system design | Durable state | Principal runtime surfaces |
|---|---|---|---|---|
| **Flip** | Community identity, rooms, chat, forums, media, moderation, and AI participation. | In-product AI participant runtime with explicit triggers, actor-scoped context, admitted tools, model routing, citations, artifacts, and asynchronous continuation. | Accounts, memberships, messages, threads, synthesis runs, AI activity, tool calls, citations, artifacts, and source relationships. | Phoenix web/API, LiveView, Channels, Oban, Electric-synchronized native clients. |
| **Project Manager** | Research and technical-project planning, evidence, decisions, review, and handoff. | Structured MCP workflows externalize project memory, require causal provenance for important updates, assemble active context, and expose deterministic next work. | Projects, phases, experiments, findings, hypotheses, decisions, literature, constraints, principles, feedback, sessions, and typed edges. | Rust CLI, MCP stdio server, embedded web dashboard, versioned SQLite store. |
| **Baton** | Persistent coding-harness coordination across providers and repositories. | Run, wave, and workflow execution across full harness sessions with routing, interaction, steering, shared context, recovery, harvesting, review, and integration. | Goals, plans, workflows, routes, sessions, events, interactions, knowledge horizons, worktrees, artifacts, evidence, and result state. | Embedded API, resident application bus, CLI, MCP, provider adapters, Git worktrees. |
| **HomeCloud** | Local and remote inference, finite accelerator capacity, agent execution, research, and application services. | Recoverable agent runtime with context assembly, dynamic tools, model and backend routing, sandboxed execution, loop control, checkpoints, and lower-priority research work. | Model instances, GPU claims, queues, agent tasks, checkpoints, research records, tool and telemetry state, documents, and application objects. | Elixir/OTP application, Phoenix/LiveView, Ash/PostgreSQL, local model services, remote adapters, containers, connectors. |

## Flip: product-owned AI behavior

Flip places AI execution inside the community product rather than behind an unrestricted assistant endpoint. The product controls:

- which visible user action triggers an AI turn;
- which actor, room, reply chain, and community state are eligible context;
- which tools and effects are admitted for the current surface;
- how provider calls, tool results, citations, artifacts, and failures are recorded;
- how a generated reply becomes ordinary durable message state;
- how selected conversation enters a separate synthesis workflow and becomes a forum object with source lineage;
- how durable changes reach web and native clients while presence and typing remain ephemeral.

This project demonstrates AI product engineering at the point where model execution intersects with social identity, authorization, real-time state, background work, and publication.

## Project Manager: external project state for long-running work

Project Manager treats project continuity as an application-state problem rather than a prompt-length problem. Its typed model separates planning, execution, evidence, interpretation, and review:

- phases and dependencies determine which work is actionable;
- experiments produce findings;
- findings support or contradict hypotheses and inform decisions;
- literature, principles, constraints, research, and feedback remain connected to the work that used them;
- sessions record the active experiment, temporal changes, and handoff state;
- retrieval returns typed project context and graph neighborhoods rather than unstructured transcript recall;
- review, audit, and repair surfaces expose unsupported decisions, orphaned nodes, expired constraints, contradictory evidence, and unresolved branches.

The agent interface is intentionally stricter than raw storage access. MCP operations validate lifecycles and causal relationships before committing state, while the deterministic DAG remains independently testable.

## Baton: cross-harness fleet execution

Baton operates above vendor-native coding harnesses without reducing them to one-shot model calls. Its application model covers:

- goal, plan, and declarative workflow construction;
- exact harness, model, and effort routing;
- persistent worker sessions and provider capability adapters;
- parallel waves with member-specific scopes and progress;
- questions, approvals, decision requests, replies, checkpoints, nudges, and recovery;
- task-, workflow-, and project-level collaboration state;
- isolated Git worktrees and content-addressed result materialization;
- verification, independent review, adoption, and integration as later stages of the run lifecycle.

A plain-code coordinator carries event ordering, liveness, capacity, version fences, replay, and cleanup beneath the AI orchestrator. This division allows the orchestrator to plan and steer while durable execution state remains available to every control surface.

## HomeCloud: resource-aware AI execution

HomeCloud connects model and agent behavior to actual infrastructure state. Its runtime coordinates:

- local and remote model profiles;
- healthy inference instances, slot capacity, priority, and prompt-cache affinity;
- GPU claims, workload queues, model-service lifecycle, cooldowns, and baseline return;
- autonomous, interactive, and plan-only agent modes;
- per-task sandboxes, durable workspaces, tool discovery, and tool execution;
- loop and quality-regression detection;
- phase-aware checkpoints containing message, plan, event, loop, and engine state;
- lower-priority research, evaluation, OCR, media, connector, and maintenance workloads.

The result is an application runtime rather than a collection of independently managed inference servers. Interactive requests, agents, and background programs share one supervised resource and execution model.

## System boundaries

| Boundary | Implemented relationship |
|---|---|
| **Flip and inference infrastructure** | Flip owns product authorization, context, tools, effects, and durable content. It can use hosted or local provider-compatible inference without depending on HomeCloud. |
| **Project Manager and agent runtimes** | Project Manager exposes structured project state through CLI, MCP, and web interfaces. It can support human or agent work without requiring Baton or HomeCloud. |
| **Baton and coding harnesses** | Baton coordinates full external harness sessions through adapters. Harness-specific capabilities remain visible; Baton does not assume identical controls across providers. |
| **HomeCloud and application products** | HomeCloud supplies local-first inference and execution services. Applications retain their own domain authorization and product semantics. |
| **Cross-project use** | Standards-based integration is possible, but the portfolio does not claim a deployed dependency graph among the four projects. |

## Implementation availability

Project Manager and Baton have public implementation repositories. Flip and HomeCloud remain private systems and are documented through their public case studies, architecture diagrams, source-path evidence, and synthetic or product-facing demonstrations.

[← Back to portfolio](README.md)
