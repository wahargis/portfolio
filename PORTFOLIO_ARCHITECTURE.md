# System Boundaries and Interfaces

The portfolio contains four independent systems. They can exchange standards-based data or services, but none is a required component of another.

## Responsibility boundaries

| System | Owns | Does not own |
|---|---|---|
| **Flip** | Community identity, membership, chat, forums, AI participation, media, citations, artifacts, synthesis, and user-facing authorization | Model-process scheduling, coding-agent delivery, or general project/research management |
| **Project Manager** | Long-running project state: phases, experiments, findings, hypotheses, decisions, literature, constraints, review, retrieval, and session continuity | Repository mutation, model hosting, or community-product state |
| **Baton** | Delegated software work: runs, plans, exact routes, persistent harness sessions, waves, interactions, workspaces, verification, review, and integration | The internal reasoning loop of each coding harness or the application state of the repository being changed |
| **HomeCloud** | Local/remote inference routing, model instances, GPU capacity, agent execution, sandboxes, checkpoints, research workloads, and shared AI application services | Flip’s social authority, Project Manager’s project record, or Baton’s repository-delivery lifecycle |

## Control surfaces and authoritative state

### Flip

Flip exposes product behavior through Phoenix web, LiveView, Channels, HTTP APIs, synchronized React clients, and background workers. PostgreSQL is authoritative for user-visible state. Oban records durable asynchronous execution. Provider responses become product state only after the relevant domain transaction succeeds.

### Project Manager

Project Manager exposes one SQLite-backed project model through a Rust CLI, an MCP stdio server, and an embedded browser dashboard. The CLI supports direct administration and scripting. MCP adds structured causal and lifecycle constraints for agent-driven work. The browser presents portfolio, graph, phase, and review views over the same store.

### Baton

Baton exposes embedded application calls, an authenticated resident bus, CLI, web, and MCP surfaces. These surfaces operate over one run application rather than independent controllers. The coordinator kernel owns event ordering, replay, process lifecycle, liveness, capacity, and verification state; harness adapters translate provider-native session controls.

### HomeCloud

HomeCloud exposes Phoenix application surfaces, connectors, internal service calls, and model-compatible endpoints. OTP supervision owns process lifecycle. PostgreSQL/Ash persists application and execution records. Instance pools, GPU claims, and workload scheduling determine which local capacity is actually available.

## Integration surfaces

### Flip and HomeCloud

Flip can route an AI turn to a HomeCloud-hosted OpenAI-compatible endpoint. HomeCloud owns model and accelerator operations; Flip still owns the invoking actor, room visibility, tool catalog, citations, artifacts, and persisted reply.

### Baton and project repositories

Baton can coordinate work against any repository for which it has an admitted workspace and route. A repository does not need to adopt Baton’s internal data model. Baton’s output is a captured, reviewable result tied to the run that produced it; verification can be required by the admitted run or workflow policy.

### Project Manager and agent workflows

A person or agent can use Project Manager to preserve the reasoning and execution state surrounding work performed elsewhere. Its MCP tools can guide a long-running investigation, but Project Manager does not execute arbitrary repository changes or allocate inference hardware.

### HomeCloud and agent applications

HomeCloud can provide inference, tool execution, research, OCR, document, and media services to local applications. Those applications retain their own product authority. HomeCloud decides how finite compute and execution environments are operated.

## Lifecycle and state diagrams

| Project | Diagram | Scope |
|---|---|---|
| Flip | [Conversation-to-forum lifecycle](flip-technical-overview/diagrams/conversation-to-forum.svg) | Synthesis admission, planning, operation ledger, forum commit, source lineage, linkback, and recuration |
| Project Manager | [Typed project record](project-manager/assets/project-record.svg) | Phase DAG, experiments, findings, hypotheses, decisions, constraints, sessions, and review |
| Baton | [Run lifecycle](baton/assets/run-lifecycle.svg) | Admission, allocation, persistent execution, interaction, recovery, capture, verification, review, integration, and closure |
| HomeCloud | [Logical platform architecture](homecloud/assets/diagrams/architecture.svg) | Admission, model capacity, GPU authority, agent execution, checkpoints, persistence, and application services |
| HomeCloud | [Reference deployment](homecloud/assets/diagrams/reference-deployment.svg) | Dated accelerator, application, storage, and external-provider deployment |

## Source availability

Project Manager and Baton have public implementation repositories. Flip and HomeCloud are private implementations; their portfolio pages cite internal module paths, persisted entities, and execution contracts without linking to inaccessible repositories or exposing production data and configuration.

[Back to the portfolio](README.md)
