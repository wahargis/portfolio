# Baton

Baton is a fleet controller for persistent coding-agent sessions. It coordinates native coding harnesses from several providers through one run model while preserving each harness's session protocol, model choices, approvals, questions, interruption behavior, usage reporting, and event stream.

A Baton run can decompose an approved software objective into dependency-ordered work, assign exact harness, model, and effort routes, operate several workers over many turns, carry shared context and results between them, isolate repository changes, and keep the operator involved at decisions, approvals, waits, and checkpoints. The result is a controlled multi-agent software process rather than a set of unrelated model calls.

## System scope

| Area | Current system |
|---|---|
| **Run application** | Goal and plan compilation, distinct plan approval, task topology, run status and wait, answers, steering, stop, recovery, evidence, review, adoption, and integration operations. |
| **Fleet routing** | Exact harness, model, and effort tuples; provider capability records; live route state; policy and budget constraints; provider-specific persistent-session adapters. |
| **Orchestration** | Dependency-ready scheduling, waves, workflow definitions and interpretation, bounded concurrency, task and resource ownership, and run settlement. |
| **Interaction and attention** | Questions, approvals, decisions, dependency waits, turn checkpoints, work progress, interruption, steering, operator attention, and terminal session state. |
| **Coordination and context** | Append-only events, run projections, messages, shared scratch and knowledge, context maps and programs, predecessor results, and handoff between workers. |
| **Repository and result handling** | Git worktrees, path scope, workspace capacity, capture, clean verification workspaces, structured merge support, result lineage, evidence, review, adoption, integration, export, and cleanup. |
| **Control surfaces** | Resident application authority with shared CLI, MCP, and web operations and consistent run semantics. |

## Architecture

<img src="assets/fleet-driver.svg" alt="Baton run application, fleet routing, persistent sessions, coordination state, and repository results" width="1100" />

The diagram separates the run application from the native coding harnesses. Baton schedules and controls the work around each session. The harness remains responsible for its model-native reasoning and tool loop.

## Run lifecycle

A representative run proceeds through the following stages:

1. **Create a goal.** The operator or orchestrator supplies the software outcome, definition of done, constraints, risk, repository scope, and bounded time, token, cost, and provider-turn budgets.
2. **Compile and approve a plan.** Baton creates dependency-ordered nodes with expected effects, allowed paths, route policy, predecessor requirements, and verification commands. Planning and execution are separate operations; starting a run does not silently dispatch workers before plan approval.
3. **Admit the run.** The application validates the goal and plan, records canonical versions and digests, checks route and scope constraints, and creates the durable task topology.
4. **Open ready work.** The scheduler identifies dependency-satisfied nodes, applies concurrency and resource limits, and starts them as a wave or through a stored workflow.
5. **Resolve exact routes.** Capability and liveness checks select an allowed harness, model, and effort tuple. Baton does not assume that two harnesses expose the same controls because they can both edit code.
6. **Start persistent sessions.** Provider adapters create long-lived coding-harness sessions in Baton-owned workspaces. The worker receives the approved objective, local scope, predecessor context, expected effects, and completion contract.
7. **Supervise work and interactions.** Events update the run projection. Questions, approvals, decisions, waits, checkpoints, stalls, interrupts, and steering remain structured state. Baton can continue the same native session rather than replacing every action with a new prompt.
8. **Capture candidate results.** Repository changes are captured from the owned worktree and associated with the exact run, task, route, session, base, and plan version that produced them.
9. **Evaluate and preserve results.** Verification and optional independent review operate over the captured candidate. A successful result can be preserved and selected without immediately merging it.
10. **Adopt or integrate deliberately.** Adoption records the selected candidate. Integration applies an allowed local repository operation after the required evidence and review checks. Publication or deployment is not implied by worker completion.
11. **Settle and recover resources.** Sessions, worktrees, leases, interactions, events, results, and cleanup state remain available for inspection and recovery.

## Shared command and control model

Baton's ordinary CLI, MCP, and web operations use one run application. The resident application process owns the active fleet and authoritative run state. A one-shot client does not become a second controller merely because it can issue a command.

The run interface includes bounded operations for status, waiting, answering a specific interaction, steering a specific run member, stopping one run, retrieving evidence, selecting a candidate result, requesting review, and applying an allowed integration. Commands identify the run and current target state and can be rejected when the target has advanced or no longer belongs to the caller's current view.

This shared model is important for agent callers. An orchestrator using MCP and a person using the web interface can inspect the same run, but the system still decides which interaction can be answered, which worker is current, and whether an operation is stale or already completed.

## Goal, plan, and task topology

Goals and plans are validated data rather than a prompt transcript.

A goal records the requested outcome, definition of done, constraints, risk, and budgets. A plan records nodes, dependencies, repository scope, expected effects, route restrictions, predecessor evidence, and verification. Canonical serialization and digests give later events and results a stable reference to the version that was approved.

The task topology determines which nodes can run concurrently and which must wait. Dependencies are explicit. The scheduler does not treat a worker's informal claim that another task is complete as a dependency transition. Completion, failure, cancellation, or preserved partial result is recorded by the run application.

Goal or plan revisions are versioned. Later revisions cannot silently weaken established constraints or change the meaning of evidence already attached to an earlier plan.

## Route and provider engineering

A Baton route is an exact harness, model, and effort tuple. Route admission considers:

- whether the harness adapter is available and healthy;
- whether the requested model and effort are supported by that harness;
- whether the adapter can provide the required interaction, interruption, streaming, and usage behavior;
- live concurrency and provider limits;
- run-level route policy, risk, budget, and fallback restrictions.

Provider adapters expose a common session-shaped contract while retaining provider differences. The contract covers session creation, prompt or turn submission, incremental events, questions and approvals, steering, interrupt, stop, result capture, and recovery. Capability records state which parts are native, emulated, unavailable, or ambiguous.

This avoids two common errors: routing work to a nominal model name that cannot be started on the selected harness, and treating command acknowledgement as proof that a provider session reached the requested state.

## Waves, workflows, and persistent sessions

A wave groups work that can proceed under the current dependency and resource state. The wave driver updates each member from provider events and run interactions, applies bounded corrective behavior, and settles when members reach terminal states.

Status is derived from the type and meaning of current state. A worker waiting for a predecessor is not reported as actively working. A question or approval blocks unattended progression. A turn checkpoint may permit a nudge or redirect, while a provider process that has not acknowledged stop remains active.

Stored workflows support repeated multi-stage coordination patterns. Workflow definitions, policies, revisions, and the interpreter turn higher-level process structure into run operations without giving workers direct authority over the fleet. Workflows can sequence roles, fan work out, collect results, request review, or route a later stage based on recorded state.

The underlying sessions remain native coding-harness sessions. Baton does not reimplement every provider's coding loop as raw API calls. This preserves mature harness behavior such as code navigation, tool use, local context management, and provider-specific interaction while making the surrounding multi-agent process consistent.

## Interaction, interruption, and attention

Questions, approvals, decisions, waits, and checkpoints are part of execution state.

A worker can ask for information instead of guessing. A provider can request approval for an operation. A task can wait for another node. The operator can nudge a worker at a natural boundary, redirect the current turn when supported, or request interruption. These states are represented separately because they require different system behavior.

An answer is applied to one current interaction. Competing attempts to answer an already resolved interaction are rejected. Interruption uses the strongest semantics the provider exposes and remains pending until the event stream confirms the resulting lifecycle state. When a provider can only emulate steering by interrupting and starting another turn, the adapter reports that behavior instead of presenting it as equivalent to native in-turn redirection.

The run view summarizes which members are working, waiting, blocked, asking for input, approaching limits, stopped, failed, or complete. Raw events remain available for audit and recovery, but the operator is not required to infer the fleet state from interleaved provider text.

## Shared context and coordination state

Baton maintains context outside individual model sessions. This includes the approved goal and plan, worker briefs, task topology, predecessor results, shared messages, scratch state, project knowledge, capability records, interaction state, and run events.

Workers do not gain direct authority over one another through unrestricted chat. Coordination is mediated through stored context and application operations. This keeps handoff inspectable and allows the controller to determine which task, run, or result a message belongs to.

Context maps and programs support repeatable selection and transformation of run state for a worker or stage. They can assemble the relevant goal, plan node, predecessor evidence, repository information, and shared findings without copying the entire fleet log into every session.

The coordination store and event model also support restart. Run projections can be reconstructed from durable facts instead of relying on one in-memory conversation or process tree.

## Repository isolation and resources

Each coding worker operates in a Baton-owned Git worktree pinned to a known base. Path scope, workspace ownership, capacity, and cleanup are explicit controller state.

Worktrees provide more than parallel directories. They define which controller owns the physical workspace, which base and candidate commit are associated with a task, and which state is evaluated. A worker can use its normal coding tools inside the workspace, but it does not gain authority to publish arbitrary host paths or adopt its own result.

Repository handling includes path-escape and symlink checks, ownership metadata, reconciliation after restart, capacity limits, candidate capture, clean verification workspaces, structured merge support, and cleanup. Conflicting or ambiguous state fails before integration rather than being normalized into a successful result.

Provider sessions, worktrees, route slots, and other finite resources are allocated together. A scheduler should not dispatch a worker only to discover that its repository workspace or provider capacity is unavailable.

## Results, evidence, review, and integration

Worker output is a candidate result. The result record ties the candidate to the run, approved plan, task node, base, workspace, route, session lineage, observed effects, and verification evidence.

Verification can run the plan's specified checks in a separate workspace at the captured candidate state. Independent review can use another admitted route and receives the immutable result and relevant contract rather than the worker's unverified summary. These operations inform result selection; they do not redefine the original goal or plan.

Adoption records which preserved candidate is selected. Integration is a separate operation that applies an allowed local change after checking result identity, evidence, review, repository state, and current policy. This separation permits several candidates or reviews to exist without treating the first worker to finish as the accepted repository state.

Result export uses controlled roots and ownership checks. Ambiguous target ownership, replacement of an existing result, or a mismatch between the requested result and recorded lineage is rejected.

## Recovery and failure handling

| Problem | System behavior |
|---|---|
| Controller restart | Durable run, session, interaction, workspace, lease, event, and result state is reconciled before new effects are admitted. |
| Stale command | Run, target, version, or fence mismatch rejects a command that no longer applies. |
| Provider disconnect | Adapter and recovery state distinguishes a lost transport from a confirmed stopped or terminal session. |
| Unanswered question or approval | The interaction remains visible and blocks progression that requires the answer. |
| Legitimate dependency wait | The member is represented as waiting rather than stalled and is not repeatedly prompted to continue. |
| Unproductive loop or stalled member | Bounded corrective actions, route policy, or explicit finalization can stop indefinite operation. |
| Dirty or missing workspace | Worktree ownership and repository checks fail before capture, verification, adoption, or integration. |
| Worker claims completion without evidence | The claim is stored as worker output; result and verification state remain separate. |
| Conflicting candidate results | Candidates remain distinct and can be reviewed or selected explicitly. |
| Ambiguous export or integration target | The operation fails closed without overwriting or publishing uncertain state. |

## Public implementation

The implementation is available in the **[Baton repository](https://github.com/Flip-Engineering/baton)**.

| Source area | Engineering content |
|---|---|
| [`impl/src/application.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/application.mjs) and [`application-semantics.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/application-semantics.mjs) | Run operations, lifecycle transitions, result selection, review, adoption, integration, and shared application semantics. |
| [`impl/src/goal-plan.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/goal-plan.mjs) and [`orchestrator-plan.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/orchestrator-plan.mjs) | Goal and plan validation, canonical versions, budgets, dependencies, route policy, path scope, and completion contracts. |
| [`impl/src/coordinator.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/coordinator.mjs) and [`coordination-store.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/coordination-store.mjs) | Resident fleet coordination, durable events and projections, interactions, sessions, resources, recovery, and shared state. |
| [`impl/src/adapter.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/adapter.mjs), [`router.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/router.mjs), and [`route-liveness.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/route-liveness.mjs) | Persistent-session contract, provider capabilities, exact route selection, and live availability. |
| [`impl/src/wave-driver.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/wave-driver.mjs) and [`workflow-interpreter.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/workflow-interpreter.mjs) | Multi-member supervision, interaction precedence, waiting and progress state, workflow execution, and settlement. |
| [`impl/src/messages.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/messages.mjs), [`context-runtime.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/context-runtime.mjs), and [`context-program.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/context-program.mjs) | Stored coordination messages, shared context, context selection and transformation, and result handoff. |
| [`impl/src/worktree.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/worktree.mjs) and [`worktree-capacity.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/worktree-capacity.mjs) | Git workspace ownership, capacity, scope, capture, clean verification, reconciliation, and cleanup. |
| [`impl/src/run-timeline.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/run-timeline.mjs), [`run-lineage.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/run-lineage.mjs), and [`result-export.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/result-export.mjs) | Operator-readable run history, result lineage, evidence association, and controlled export. |
| [`impl/src/mcp-northbound.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/mcp-northbound.mjs) and [`web-northbound.mjs`](https://github.com/Flip-Engineering/baton/blob/master/impl/src/web-northbound.mjs) | MCP and web access to the resident run and fleet application. |

## Current boundaries

Baton does not replace the reasoning loop of each coding harness, and a shared adapter does not imply identical provider behavior. Route capability and liveness remain explicit.

The controller can govern commands and repository effects that pass through its application and workspace model. It cannot retroactively undo an external side effect that a worker completed outside those controls. Such effects require separate policy and approval handling.

A successful run result is not automatically published or deployed. Adoption, local integration, pushing, release, and deployment remain distinct operations with separate authority.

[Back to the portfolio](../README.md)
