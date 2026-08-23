# Agent Engineering and AI Systems Design

The four projects in this portfolio are independent systems. They address different parts of agent engineering: operation inside a product, durable project reasoning, coordination of several coding agents, and execution on self-hosted AI infrastructure.

## System comparison

| Project | Input to the system | Work performed by models or agents | Control retained by software | Durable state | Recovery path |
|---|---|---|---|---|---|
| **Flip** | A message, reply, curation request, document or data task, media request, or completed asynchronous artifact. | Interpret the request, select admitted retrieval or tools, analyze evidence, compose a reply, create an artifact, or propose a product action. | Identity, membership and visibility, context eligibility, tool catalog, trusted retrieval scope, effect schemas, provider credentials, timeouts, output validation, persistence, and publication. | Messages, forum objects, AI activity, tool calls, citations, artifacts, synthesis runs, jobs, and client-visible lifecycle state. | Unique jobs, bounded retry and repair, stale-work recovery, provider fallback under route policy, durable failed artifacts, and one continuation from terminal asynchronous events. |
| **Project Manager** | Findings, experiments, hypotheses, decisions, constraints, literature, phases, reviews, handoffs, and session updates. | Assist with retrieval, contradiction review, confidence analysis, orientation, and structured updates to project state. | Typed node and edge vocabulary, validation, causal relationships, branch and merge topology, deterministic phase dependencies, persistence, and interface semantics. | Project objects and edges, temporal changes, review results, session records, phase state, and retrieval context. | Rebuild session context from stored project state, retain superseded and contradictory history, detect invalid graph or DAG state, and continue from dependency-satisfied phases. |
| **Baton** | An approved software objective, plan, repository scope, route policy, budgets, and verification requirements. | Coding agents inspect, edit, test, explain, ask questions, and produce candidate results through their native harnesses. | Goal and plan validation, route admission, worker lifecycle, task dependencies, interaction handling, worktree ownership, event ordering, interruption, verification, result selection, adoption, and integration. | Runs, plans, task topology, routes, sessions, events, questions, approvals, waits, checkpoints, shared context, workspaces, results, and evidence. | Event replay, fencing of stale commands, session and worktree reconciliation, bounded corrective actions, explicit stopped or failed state, and preserved candidate results. |
| **HomeCloud** | Interactive inference, agent tasks, research jobs, document work, connector activity, and multimodal workloads. | Models plan, reason, select tools, generate or analyze content, and continue through multi-turn execution. | Model routing, instance checkout, workload priority, GPU claims, model-service transitions, sandbox allocation, tool profiles, loop limits, checkpointing, cleanup, health, and persistence. | Model-instance state, queues, GPU ownership, agent tasks, messages, plans, tool results, checkpoints, research records, artifacts, and service health. | OTP restart, instance health and restart state, scheduler reconciliation, container cleanup, phase-aware checkpoints, resumable agent state, and controlled return to baseline capacity. |

## Flip execution path

```text
product event
  -> actor, community, room or forum, and AI identity are resolved
  -> bounded product context and permitted retrieval are selected
  -> a route and turn-specific capability catalog are admitted
  -> the model can retrieve, analyze, call tools, or request an effect
  -> citations, actions, and artifacts receive durable identities
  -> the terminal reply or result is validated and committed
  -> web and native clients receive durable and realtime updates
```

The agent is operating inside the product's identity, authorization, data, and lifecycle rules. The server does not provide a generic database or unrestricted product API to the model. Internal retrieval carries trusted actor and community scope. Effectful operations use domain services and idempotency appropriate to the action. Long-running media or document operations can finish after the original model turn and resume the product workflow through a new, deduplicated continuation.

## Project Manager execution path

```text
session, experiment, review, or decision
  -> typed project objects and relationships are written
  -> causal, support, contradiction, supersession, and dependency links are retained
  -> retrieval and review assemble the relevant project state
  -> phase readiness is computed from the execution DAG
  -> the next person or agent starts from the stored state and current dependencies
```

The project record is not limited to notes or task status. It retains the relationships required to explain why a decision exists, which experiment produced a finding, what changed after a branch or review, and which active work depends on a disputed result. Model-assisted review is used for language interpretation. Scheduling and graph integrity remain deterministic and testable.

## Baton execution path

```text
approved goal and plan
  -> run application validates scope, dependencies, budgets, and route policy
  -> scheduler opens dependency-ready nodes as waves or workflows
  -> capability and liveness checks select exact harness/model/effort routes
  -> persistent coding-harness sessions work in isolated repository copies
  -> questions, approvals, waits, checkpoints, events, and shared context update the run
  -> results are captured, verified, reviewed, and preserved
  -> adoption and integration remain explicit controller operations
```

Baton coordinates native coding-agent sessions over many turns. It does not replace each harness's reasoning loop. Provider adapters preserve differences in approvals, questions, interruption, usage, and event semantics. The controller maintains run state outside any one model context and separates worker capability from repository, lifecycle, and result authority.

## HomeCloud execution path

```text
application, agent, research, document, connector, or media request
  -> route and priority are resolved
  -> a healthy model instance and permitted GPU capacity are acquired
  -> context, tool profile, workspace, and execution mode are prepared
  -> the model and tool loop runs under time, turn, repetition, and quality controls
  -> messages, plans, events, tool results, and checkpoints are persisted
  -> the result returns to an application service and capacity is reconciled
```

HomeCloud connects model execution to physical resource state. A local endpoint is available only when its process, instance slot, GPU claim, and queue state permit it. Agent execution uses containerized workspaces and phase-aware checkpoints. The runtime can also route to remote providers where configuration and workload policy allow, without treating local and remote capacity as operationally identical.

## Control and state boundaries

| Boundary | Flip | Project Manager | Baton | HomeCloud |
|---|---|---|---|---|
| **Identity and scope** | Product actor, community, room or forum, AI identity, and current visibility. | Project, portfolio, branch, session, node and edge types, and interface permissions. | Run, plan digest, task node, route tuple, worker session, repository scope, and controller fence. | Request owner, workload class, model route, tool profile, workspace, GPU claim, and service configuration. |
| **Context** | Selected conversation, reply ancestry, authorized internal records, external evidence, documents, and artifact state. | Connected project objects, causal and temporal neighbors, phase state, review results, and prior session handoff. | Approved goal and plan, worker brief, predecessor results, shared scratch and knowledge, interactions, and run events. | Stored messages and plans, application records, retrieved material, tool results, model limits, and checkpoint state. |
| **Effects** | Messages, forum changes, polls, citations, generated media, files, and other product-native actions. | Typed project updates, relationships, review records, phase transitions, and session state. | Worker commands, repository mutation inside owned worktrees, results, evidence, adoption, review, and integration. | Tool execution, model-process changes, GPU allocation, files in sandboxes, application records, connectors, and media operations. |
| **Failure** | No protected retrieval without valid scope; no visible effect without a committed product record. | Invalid graph or phase state is rejected; conflicting evidence is retained rather than overwritten. | Stale or ambiguous commands are refused; worker completion is separate from controller acceptance. | Unhealthy instances and disputed GPU ownership are unavailable; incomplete agent work remains resumable or explicitly failed. |

## Interfaces

The projects expose different interfaces because their users and control requirements differ:

- Flip uses Phoenix web, realtime channels, background jobs, APIs, and a React-based desktop and mobile client.
- Project Manager exposes one Rust application model through CLI, MCP, and web interfaces.
- Baton exposes a shared run and fleet-control model through CLI, MCP, web, and resident-controller paths.
- HomeCloud exposes Phoenix application surfaces and internal services over an OTP runtime that also serves other local applications and agents.

Interface code is kept outside the authoritative state transitions where practical. This prevents a CLI command, MCP tool, web route, background worker, or native client from creating a weaker parallel version of the system's rules.

## Project relationships

The projects can exchange services or records, but none requires the others to operate:

- Flip can use hosted inference or a HomeCloud endpoint while retaining its own product permissions, context, tools, and persistence.
- Baton can use local or hosted coding-agent routes without depending on HomeCloud's internal GPU scheduler.
- Project Manager can receive selected findings, decisions, or handoffs from agent work without becoming Baton's process controller.
- HomeCloud can execute agent and research workloads without adopting Flip's social model or Project Manager's project ontology.

[Back to the portfolio](README.md)
