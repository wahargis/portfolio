# Baton

> A control plane for coordinated software work across persistent coding-agent harnesses.

## At a glance

| | |
|---|---|
| **Product** | A fleet-level orchestrator that turns a software objective into scoped, supervised, verifiable work across multiple coding-agent runtimes. |
| **Users** | Engineers and operators who want to delegate substantial repository work without losing control of intent, interaction, workspace ownership, verification, or result adoption. |
| **Core problem** | Coding harnesses differ in session protocol, model selection, tool behavior, approvals, usage reporting, and failure modes. Parallel workers increase throughput but also increase ambiguity, conflicting writes, hidden blocking, and unverified completion. |
| **Engineering focus** | Goal and plan contracts, capability-aware routing, persistent sessions, human-in-the-loop interactions, isolated Git worktrees, run lineage, verification, and secure result export. |
| **Primary implementation** | Node.js control plane with CLI, MCP, and web surfaces; provider-specific persistent-session adapters; real Git worktree and verification mechanics. |
| **Source** | [`wahargis/baton`](https://github.com/wahargis/baton) |

## The product problem

A coding agent can often make a useful change. A software-delivery system must answer a larger set of questions:

- What objective and definition of done were actually approved?
- Which constraints, effects, paths, risks, and budgets apply?
- Which harness and model can perform the work with the required interaction semantics?
- Can several workers operate without sharing an uncontrolled working tree?
- Is a worker waiting, blocked on a decision, asking for approval, at a steerable checkpoint, or genuinely making progress?
- What happens when a provider session disconnects or a controller restarts?
- Who decides that a change is complete?
- Which verification evidence applies to the exact artifact being adopted?
- How is a result exported without allowing a worker to overwrite arbitrary host state?

Baton is built around those control questions. It does not treat orchestration as “send the same prompt to several models and concatenate the answers.” It maintains a durable relationship between intent, route, session, workspace, interaction, evidence, and result.

The coding harness remains responsible for model-native reasoning and tool use. Baton is responsible for the software-delivery contract around that work.

## Representative workflow

A controlled Baton run proceeds through a sequence like this:

1. **Define the goal.** The operator supplies an objective, definition of done, constraints, risk class, and bounded token, cost, time, and provider-turn budgets.
2. **Approve a plan.** Work is decomposed into dependency-ordered nodes with explicit repository scope, required effects, route allowlists, and verification commands.
3. **Validate before execution.** Baton canonicalizes and digests the goal and plan, rejects unknown fields and credential-shaped content, prevents successor goals from weakening established constraints, and verifies that paths and commands remain repository-scoped.
4. **Select an exact route.** Harness, model, and effort are matched against provider capability cards and live route state instead of being assumed from a display name.
5. **Allocate isolated work.** Each worker receives a Baton-owned Git worktree pinned to a known base. The brief states its immutable context, allowed paths, mutation authority, constraints, definition of done, and exact verification contract.
6. **Supervise the persistent session.** The worker can continue over many turns. Baton distinguishes work from waiting, decisions, questions, approvals, checkpoints, and terminal state. Steering occurs at controlled interaction points rather than by replacing the session.
7. **Capture and verify the artifact.** Changes are captured from the worker workspace and can be evaluated in a fresh verification sandbox. The worker’s statement of completion is evidence, not acceptance authority.
8. **Publish a result deliberately.** Timelines, lineage, verification, and result artifacts can be exported through Baton-owned paths and no-replace publication mechanics.

This workflow is the project’s value. It lets an engineer use capable coding agents while retaining the properties expected from a serious software-delivery process.

## Architecture

<img src="assets/fleet-driver.svg" alt="Baton controlled software-delivery architecture" width="1050" />

### 1. Intent and governance

Baton represents the approved objective and plan as validated data. Goals include a definition of done, constraints, risk, and budgets. Plans include node dependencies, path scope, expected effects, permitted route tuples, and exact verification commands.

Canonical serialization and digests give later stages a stable reference to the approved contract. Goal amendments cannot silently remove an established constraint, lower risk, or expand a budget. Secret-shaped content is rejected before it becomes durable control-plane state.

This layer prevents a common orchestration failure: a prompt mutates gradually over several sessions until nobody can identify the original obligation.

### 2. Provider capability and persistent-session adapters

Different coding harnesses expose different native behavior. Some support structured approvals, some support free-form questions, some emit reliable usage, some distinguish interrupt acknowledgment from confirmed stop, and some provide richer model-routing metadata.

Baton uses a shared session-shaped adapter contract while preserving those differences in capability cards. The contract covers session spawn, prompting and steering, interrupt, approval, answers, kill, and event delivery. It does not force every provider into a fictional least-common-denominator one-shot call.

The route is an explicit tuple—harness, model, and effort—and must match a live, unambiguous capability. This makes routing inspectable and prevents a nominally valid configuration from spawning work onto an unavailable or incompatible provider path.

### 3. Supervised runs and waves

A run can contain multiple persistent members operating over a long period. The wave driver polls state, responds to blocking interactions, steers at checkpoints, detects stalls from semantic progress rather than global event noise, and settles the wave when members reach terminal state.

The status reducer is important. A worker waiting on another unit is not rendered as “working.” A pending decision or approval blocks automated nudge and claim behavior. A checkpoint with a claim is different from a checkpoint without one. These distinctions keep the operator’s view and the controller’s behavior aligned.

Baton also treats control acknowledgments and lifecycle events differently. An interrupt request returning successfully is not the same as evidence that the provider process has stopped. Authoritative state comes from the event stream.

### 4. Repository isolation and ownership

Workers operate in Git worktrees created under Baton-owned repository paths and pinned to a known base commit. Verification can run in a separate throwaway sandbox rather than trusting the mutable worker directory.

The worktree layer includes typed errors, confined authority roots, path-escape and symlink checks, durable ownership metadata, reconciliation, cleanup, commit capture, and changed-line analysis. The purpose is not merely to avoid merge conflicts. It is to establish which controller owns which physical workspace and which artifact was actually evaluated.

The provider’s tool permission is therefore not the same as repository authority. A worker may have a shell, but the brief and worktree constrain where repository mutation is valid.

### 5. Verification, lineage, and result publication

Baton carries the verification contract inside the plan and renders it verbatim into the worker brief. Required predecessor evidence is part of that contract. Verification can be performed against a captured artifact in a fresh sandbox, preserving the relationship between the approved plan, the exact change, and the observed result.

Run timelines and lineage provide the operational history needed to understand how a result was reached. Export publication uses Baton-controlled roots, ownership checks, private permissions, exact structure, atomic no-replace behavior, and fail-closed recovery when ownership is ambiguous.

This is the final authority boundary: a worker can produce an artifact, but only the controller can publish it as the result of the approved run.

## Key design decisions

### Preserve native harness semantics

Baton standardizes control concepts without pretending all providers behave identically. Capability cards and adapter events make provider differences visible to routing and supervision.

This is more reliable than normalizing every provider into a text-completion abstraction and discovering semantic differences only after a run fails.

### Make goals and plans enforceable data

Objective prose alone is too weak for long-running, multi-worker delivery. Scope, effects, dependencies, route allowlists, budgets, and verification are validated fields with canonical digests.

The contract can therefore be checked by code at admission, dispatch, verification, and export.

### Treat interaction as part of execution

Questions, decisions, approvals, waits, and steerable checkpoints are not incidental messages. They determine whether a run can proceed safely. Baton models them as structured state and gives them precedence in control decisions.

### Separate worker capability from lifecycle authority

The worker cannot prove that its own interrupt completed, declare its own workspace adopted, or convert a partial result into successful publication. Lifecycle and result authority remain with the resident controller and its event/evidence record.

### Isolate parallel work physically

Separate Git worktrees reduce conflicting mutation and create a concrete unit for ownership, capture, verification, and cleanup. Parallelism is therefore bounded by available workspaces and provider capacity, not only by the number of prompts an operator can issue.

### Verify the artifact, not the narrative

A fluent completion message is not a test result. Baton carries the exact verification command and predecessor-evidence requirements, evaluates the captured state, and records the outcome separately from worker prose.

## Failure and recovery model

Baton is designed around partial and ambiguous outcomes:

- **Provider interruption:** persistent sessions can emit stop, crash, or terminal events that are distinct from command acknowledgments.
- **Blocked interaction:** decisions, questions, and approvals remain explicit and can pause automated progression.
- **Legitimate waiting:** dependency waits are represented separately from stalls, preventing destructive “keep going” nudges.
- **Unproductive loops:** wave policy can bound corrective nudges and require a finalization strategy rather than run indefinitely.
- **Controller restart or residue:** ownership records, leases, reconciliation, and recovery paths avoid treating stale process state as proof of current authority.
- **Dirty or conflicting repository state:** typed worktree and structured-merge errors fail before ambiguous adoption.
- **Unsafe export target:** result publication validates root ownership, permissions, identity, and no-replace semantics; uncertainty fails closed.
- **Plan drift:** canonical goal/plan references make it possible to reject execution or evidence that does not belong to the approved version.

## Implementation evidence

| Source | What it demonstrates |
|---|---|
| [`impl/src/goal-plan.mjs`](https://github.com/wahargis/baton/blob/master/impl/src/goal-plan.mjs) | Goal and plan schemas, canonical digests, constraint-preserving amendments, budgets, route allowlists, path scope, verification contracts, and secret-shaped input rejection. |
| [`impl/src/adapter.mjs`](https://github.com/wahargis/baton/blob/master/impl/src/adapter.mjs) | The persistent-session adapter contract, provider capability cards, authoritative event semantics, and the scoped worker brief. |
| [`impl/src/wave-driver.mjs`](https://github.com/wahargis/baton/blob/master/impl/src/wave-driver.mjs) | Long-running multi-member supervision, blocking interactions, waiting, checkpoint steering, stall detection, corrective budgets, and settlement. |
| [`impl/src/worktree.mjs`](https://github.com/wahargis/baton/blob/master/impl/src/worktree.mjs) | Real Git worktree lifecycle, ownership confinement, capture, verification sandboxes, typed failures, reconciliation, and cleanup. |
| [`impl/src/result-export.mjs`](https://github.com/wahargis/baton/blob/master/impl/src/result-export.mjs) | Private result roots, ownership and process-identity checks, atomic no-replace publication, and fail-closed recovery. |
| [`impl/src/run-timeline.mjs`](https://github.com/wahargis/baton/blob/master/impl/src/run-timeline.mjs) | Operator-readable reconstruction of run state and progress from durable control-plane events. |
| [`impl/src/mcp-northbound.mjs`](https://github.com/wahargis/baton/blob/master/impl/src/mcp-northbound.mjs) | The agent-facing control surface over the same underlying application semantics. |

## What the project demonstrates

For a general software reviewer, Baton demonstrates process control, provider abstraction without semantic erasure, event-driven lifecycle management, repository isolation, verification, durable ownership, and security-sensitive filesystem design.

For an AI technical lead, it demonstrates that multi-agent effectiveness depends on more than model quality. Context, route compatibility, interaction, recovery, scope, evidence, and adoption authority determine whether agent work can be trusted inside a software process.

For an engineering manager or recruiter, Baton is evidence of work at the system boundary between AI capability and software-delivery accountability: the point where a successful model response must become a controlled, reviewable, and reproducible engineering result.

## Scope and boundaries

- Baton is not a model provider and does not replace the native reasoning loop of each coding harness.
- It is not a free-form swarm in which workers share an uncontrolled repository and negotiate authority through chat.
- A common adapter does not imply identical provider capability; unsupported or ambiguous routes must remain visible.
- A worker result is not automatically an adopted repository change.
- This page emphasizes the control flow and engineering rationale. The repository contains substantially more internal policy, knowledge, workflow, web, MCP, deployment, and analysis machinery than is useful to enumerate here.

## Review paths

**Five minutes:** read **The product problem**, **Representative workflow**, and **Key design decisions**.

**Twenty minutes:** continue through **Architecture**, **Failure and recovery model**, and the first five source links.

**Deep review:** follow a goal and plan from validation through adapter dispatch, wave supervision, worktree capture, verification, and result export.

[← Back to portfolio](../README.md) · [View source repository](https://github.com/wahargis/baton)
