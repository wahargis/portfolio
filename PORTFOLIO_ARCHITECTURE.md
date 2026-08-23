# Portfolio System Boundaries

The portfolio contains four independent systems. They address different parts of agent engineering and AI systems design: operation inside a multi-user product, durable project reasoning, control of persistent coding-agent fleets, and execution on self-hosted AI infrastructure.

## System comparison

| Project | Primary input | Model or agent work | Software-owned control | Durable state | Recovery |
|---|---|---|---|---|---|
| **Flip** | Message, reply, research request, document or data task, product action, media request, curation request, or completed asynchronous artifact. | Interpret requests, retrieve authorized context, read external evidence, use typed tools, compose replies, create artifacts, and propose product actions. | Identity, membership, visibility, context eligibility, tool admission, trusted scope, effect schemas, provider credentials, deadlines, output validation, persistence, publication, and client delivery. | Messages, forum objects, AI activities, tool calls, citations, artifacts, synthesis runs, jobs, and client-visible lifecycle state. | Unique jobs, bounded retry, stale-work recovery, route-compatible fallback, durable failed artifacts, and deduplicated continuation from terminal asynchronous work. |
| **Project Manager** | Experiments, findings, hypotheses, decisions, constraints, literature, phases, reviews, branches, merges, handoffs, and session updates. | Assist with retrieval, contradiction review, confidence analysis, orientation, and structured project updates. | Typed node and edge vocabulary, validation, causal and temporal relationships, branch and merge topology, deterministic phase dependencies, persistence, and interface semantics. | Project objects and edges, temporal changes, review results, branch and merge state, sessions, phase state, and retrieval context. | Rebuild session context from stored project state, retain contradicted and superseded history, reject invalid graph or DAG state, and continue from dependency-satisfied phases. |
| **Baton** | Approved software goal, plan, repository scope, dependencies, route policy, budgets, required effects, and verification requirements. | Coding agents inspect, edit, test, explain, ask questions, and produce candidate results through their native harnesses. | Goal and plan validation, route admission, worker lifecycle, dependency scheduling, interaction handling, worktree ownership, event ordering, interruption, verification, result selection, adoption, integration, and cleanup. | Runs, plans, task topology, routes, sessions, events, questions, approvals, waits, checkpoints, shared context, workspaces, candidate results, and evidence. | Event replay, stale-command fencing, session and worktree reconciliation, bounded corrective actions, explicit stopped or failed state, and preserved candidate results. |
| **HomeCloud** | Interactive inference, agent tasks, research jobs, document work, connectors, browser or search activity, and multimodal workloads. | Models plan, reason, select tools, generate or analyze content, and continue through multi-turn execution. | Route selection, instance checkout, workload priority, GPU claims, model-service transitions, sandbox allocation, tool profiles, loop limits, checkpointing, cleanup, health, and persistence. | Model-instance state, queues, GPU ownership, agent tasks, messages, plans, tool results, checkpoints, research records, artifacts, and service health. | OTP restart, instance health and restart state, scheduler reconciliation, container cleanup, phase-aware checkpoints, resumable agent state, and controlled return to baseline capacity. |

## Flip

### Execution flow

```text
product event
  -> resolve actor, AI identity, community, origin, visibility, and feature state
  -> assemble bounded conversation, product, document, data, source, and artifact context
  -> select a compatible route and turn-specific capability catalog
  -> execute model and typed tool rounds
  -> validate citations, artifacts, actions, and terminal output
  -> commit the product result and AI activity
  -> deliver durable and realtime state to web and native clients
```

Long-running media, document, and other provider-backed operations create durable pending artifacts. Provider completion or failure updates the artifact and can start one deduplicated continuation under the original product scope.

### State and authority

Flip's authoritative state includes accounts, membership, rooms, messages, forums, AI identities, activities, tool calls, sources, citations, artifacts, synthesis runs, jobs, and client synchronization records. Providers receive bounded requests and return results; they do not determine product access or commit product state.

### Engineering work

Flip combines agent runtime design with application engineering: actor-scoped context, product-native tools, effect authority, cross-domain authorization, asynchronous artifact continuation, conversation curation with source provenance, and convergence across server-rendered web and React-based desktop and mobile clients.

## Project Manager

### Execution flow

```text
session, experiment, review, or decision
  -> validate typed project objects and relationships
  -> persist evidence, causal, branch, merge, temporal, and phase state
  -> retrieve the relevant project neighborhood
  -> review contradiction, confidence, stale state, and unresolved work
  -> update decisions or phase state explicitly
  -> compute dependency-ready execution phases
  -> write a durable handoff for the next session
```

### State and authority

The shared Rust application and SQLite store define the project vocabulary, graph relationships, lifecycle, branch and merge records, reviews, session state, and phase dependencies. CLI, MCP, and web interfaces call the same operations.

### Engineering work

Project Manager implements durable project memory as maintained state rather than prompt history. It supports causal traceability, evidence review, branch-aware history, graph-aware retrieval, session continuity, and deterministic phase scheduling while keeping advisory model analysis separate from committed project truth.

## Baton

### Execution flow

```text
approved goal and plan
  -> validate scope, dependencies, budgets, effects, routes, and verification
  -> open dependency-ready work as waves or workflow stages
  -> select exact harness, model, and effort routes
  -> start persistent coding-harness sessions in owned Git worktrees
  -> record questions, approvals, waits, checkpoints, events, steering, and shared context
  -> capture candidate repository states and result lineage
  -> verify, review, select, adopt, integrate, export, and clean up explicitly
```

### State and authority

Baton stores run, plan, task, route, session, event, interaction, context, workspace, candidate, evidence, review, and integration state outside any individual worker context. Provider adapters retain native harness differences in questions, approvals, interruption, usage, and event semantics.

### Engineering work

Baton implements control of persistent coding-agent fleets. It combines deterministic scheduling, capability-aware routing, structured human interaction, shared context, repository ownership, verification, and explicit result adoption without replacing each harness's native coding loop.

## HomeCloud

### Execution flow

```text
application, agent, research, document, connector, or media request
  -> classify workload, priority, modality, and route requirements
  -> acquire a healthy model instance and permitted GPU capacity
  -> prepare context, tool profile, execution mode, and workspace
  -> run model and tool turns under time, repetition, and quality limits
  -> persist messages, plans, events, tool results, checkpoints, and application output
  -> release instance, GPU claim, and workspace resources
  -> reconcile the configured baseline service
```

### State and authority

HomeCloud records model instances, queues, GPU claims, workload ownership, agent tasks, messages, plans, tool results, checkpoints, research state, artifacts, and health. OTP supervision owns process lifecycle; the instance pool and GPU scheduler own admission and physical resource state.

### Engineering work

HomeCloud combines local AI operations with application-level agent execution. It manages finite GPU capacity, model processes, priority, sandboxed tools, checkpoint recovery, research workloads, documents, connectors, browser and search services, and multimodal application features through one supervised runtime.

## Cross-project concerns

| Concern | Flip | Project Manager | Baton | HomeCloud |
|---|---|---|---|---|
| **Identity and scope** | Product actor, AI identity, community, origin object, and current access. | Portfolio, project, branch, session, typed object, and interface operation. | Run, plan digest, task node, route tuple, worker session, repository scope, and controller fence. | Workload owner, priority, route, tool profile, workspace, model instance, and GPU claim. |
| **Context** | Conversation, authorized product records, external evidence, documents, data, and artifact state. | Connected project objects, causal and temporal neighbors, branch, phase, review, and handoff state. | Approved goal and plan, worker brief, predecessor results, shared messages, scratch, knowledge, interactions, and events. | Messages, plans, application records, retrieved material, tool results, model limits, infrastructure state, and checkpoints. |
| **Effects** | Messages, forum changes, polls, files, citations, charts, generated media, and other product-native actions. | Typed project updates, relationships, reviews, branch or merge records, phase transitions, and session state. | Worker commands, repository mutation inside owned worktrees, candidate results, evidence, adoption, review, and integration. | Tool execution, model-process changes, GPU allocation, sandbox files, application records, connectors, and media operations. |
| **Failure** | Protected retrieval or effects are refused without valid scope; incomplete AI work remains explicit product state. | Invalid graph or phase state is rejected; conflicting evidence remains available for review. | Stale or ambiguous commands are refused; worker completion remains separate from controller acceptance. | Unhealthy instances and disputed GPU ownership are unavailable; incomplete work is resumable or explicitly failed. |

## Project relationships

The projects can exchange services or records, but none depends on another for its core operation:

- Flip can use hosted inference or a HomeCloud endpoint while retaining its own permissions, context, tools, effects, and persistence.
- Baton can use local or hosted coding-agent routes without depending on HomeCloud's internal GPU scheduler.
- Project Manager can receive selected findings, decisions, or handoffs from agent work without becoming Baton's process controller.
- HomeCloud can execute agents and research workloads without adopting Flip's social model or Project Manager's project ontology.

## Diagram coverage

- [Flip product runtime](flip-technical-overview/diagrams/product-runtime.svg) follows one product request through actor scope, context, tools, effects, persistence, artifacts, and delivery.
- [Flip detailed diagram index](flip-technical-overview/diagrams/README.md) covers retrieval, tool execution, curation, model routing, clients, deployment separation, and evaluation.
- [Project Manager architecture](project-manager/assets/architecture.svg) shows typed project state, review, retrieval, session continuity, and phase execution.
- [Project Manager Atlas example](project-manager/assets/atlas-example.svg) shows a concrete research cycle with experiments, findings, contradiction, decisions, branches, and phases.
- [Baton fleet architecture](baton/assets/fleet-driver.svg) shows command surfaces, controller state, native harness sessions, worktrees, and result lifecycle.
- [Baton recorded run](baton/assets/spec-wave-example.svg) shows a concrete multi-worker wave with dependency waits, tools, commits, and completed results.
- [HomeCloud runtime architecture](homecloud/assets/diagrams/architecture.svg) shows workload admission, model instances, GPU control, agents, tools, checkpoints, and result persistence.
- [HomeCloud reference deployment](homecloud/assets/diagrams/reference-deployment.svg) shows the current four-V100 deployment and live service configuration.

[Back to the portfolio](README.md)
