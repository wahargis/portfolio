# Baton

**A cross-harness fleet driver: one coding agent directs other full-session coding harnesses across vendors, with live communication, telemetry, intervention, shared coordination state, and durable handoff.**

Baton exists because the useful unit of delegation is often not a model API call. It is a complete coding harness—Claude Code, Codex, a GLM-backed harness, or another full session—with its own system behavior, tools, approvals, sandbox, context management, and resumable state.

Those harnesses are individually capable but operationally isolated. One cannot ordinarily treat several others as a coherent subordinate fleet: assign distinct work, see what each is doing, answer a blocked worker, redirect a bad approach mid-run, coordinate parallel branches, preserve shared context, and collect the results without writing bespoke glue around every campaign.

**Baton is the fleet driver that supplies that missing layer.**

![Baton fleet-driver architecture](assets/control-loop.svg)

## Product aim

The intended interaction is direct:

```text
Claude Code orchestrator
  -> Codex worker on implementation
  -> GLM worker on broad audit
  -> Claude worker on review

or

Codex orchestrator
  -> Claude and GLM workers in parallel
```

The orchestrator remains a full AI harness. It decides what outcome is needed, how work should be decomposed, which worker should take it, when to intervene, and what to do with the result. Baton sits underneath and around that orchestrator as the application that makes the fleet observable and controllable.

The product is not independent verification, a neutral “trust institution,” or a generic agent framework. Verification, routing, memory, and the reliability kernel matter because they make the fleet driver usable; none replaces the act of driving.

## Why full harnesses are first-class

A vendor harness is more than a prompt wrapper:

- its model is paired with vendor-tuned system behavior and tool interfaces;
- it owns planning, context compaction, approval, retry, and session semantics;
- it can retain and resume a working session rather than returning a stateless completion;
- it may use subscription-authenticated capacity with different economics from API-billed inference;
- it carries a native security and sandbox model that should be composed rather than discarded.

Baton therefore negotiates what each harness can actually do. It does not flatten every worker into a lowest-common-denominator chat API or pretend that an emulated control is equivalent to a native one.

## What “driving the fleet” means

### Delegate to live worker seats

A worker is a persistent harness session with an exact route, task brief, repository/worktree, scope, and lifecycle identity. The orchestrator can attach again after its own context changes or process reconnects. Delegation is not “spawn a subprocess and wait for stdout”; the worker remains an addressed participant in the run.

### Watch work as it happens

Baton normalizes enough cross-harness telemetry to answer the operational questions that matter:

- which workers are active;
- what phase or turn each is in;
- what each is waiting on;
- whether a worker is making progress, blocked, paused, failed, or complete;
- what artifacts and checkpoints exist;
- where the orchestrator needs to act.

The goal is not to force every vendor into an identical transcript. It is to provide a stable fleet view while retaining harness-specific evidence underneath.

### Communicate and steer through different channels

A subordinate worker needs two forms of interaction with different semantics.

**Communication** is cooperative and content-bearing: send the brief, ask a question, answer it, provide a follow-up, return a result. It can respect turn boundaries and must remain ordered and durable.

**Steering** is directional control: interrupt, redirect, pause, claim a checkpoint, stop, or seize a worker that is heading the wrong way. It cannot wait behind ordinary output or be reduced to a politely worded message in the same queue.

This separation is one of Baton's central design decisions. A worker raises a blocking question upward; the orchestrator retains downward intervention authority. The fleet remains legible because messaging and control do not compete for one ambiguous “chat” verb.

### Coordinate parallel work as runs, waves, and workflows

A **run** is one controlled unit of delegated work. A **wave** gives a group of workers one durable roster and outcome. A declarative **workflow** captures the recurring coordination pattern—membership, dependencies, steering policy, decision deferral, and result harvest—so a multi-worker campaign does not require a new process script every time.

The important consequence is that parallel work becomes an application object:

- workers can be started, attached, observed, messaged, stopped, and redriven as members of one effort;
- successful members remain usable when another member fails;
- an orchestrator can coordinate a heavyweight worker over a wider lower-cost row without losing the overall workflow identity;
- the same workflow can be invoked through the agent-facing, human, resident, or embedded surface.

### Coordinate through shared structure, not unlimited agent chatter

Baton keeps several tempos of state separate:

- the event stream records live operational truth;
- task/workflow state records ownership, dependencies, attention, and progress;
- git-backed artifacts and result pins carry code and deliverables;
- scratchpads, boards, cells, and briefing packs carry addressed coordination context;
- selected facts and decisions can be promoted into longer-lived project knowledge.

This avoids using transcript dumps as the fleet's memory. Workers coordinate through durable shared artifacts where possible; direct messages are reserved for addressed communication and intervention.

### Harvest results instead of merely ending processes

A worker result can include commits, diffs, files, tests, analysis, questions, and a structured summary. Baton binds that result to the worker and run identity so the orchestrator can review, compare, continue, redrive, or integrate it.

Independent verification is part of this harvest path: it helps the driver determine whether a worker's completion claim is usable. It is a supporting trust feature, not the project's center of gravity.

## Architecture

### Orchestrator seat

The user continues to work inside a preferred full harness. That harness is the orchestrator: it plans, delegates, reads fleet state, answers workers, and steers the campaign. A human can use the same control surfaces directly or take over a worker when needed.

### Fleet application

The application layer gives runs, waves, and workflows stable identities and semantics. It owns plan/approval state, membership, attention, messages, shared context, progress, result harvest, recovery, and controlled integration.

### Harness adapters

Each adapter exposes a capability card describing native, emulated, and unsupported behavior: session creation/resume, streamed events, messaging, interruption, checkpoint control, approvals, and termination. Vendor-specific protocol behavior remains in the adapter rather than leaking into the workflow model.

### Collaboration and knowledge substrate

The shared substrate lets workers and orchestrators exchange more than prose. It carries task briefs, questions, scratch observations, artifacts, briefing packs, result pins, and selectively promoted cross-run knowledge while keeping operational state distinct from long-horizon epistemic state.

### Coordinator reliability kernel

A plain-code coordinator carries out the orchestrator's decisions. It owns command ordering, identity fencing, durable events, replay, capacity, process ownership, recovery, and cleanup. This is plumbing under the fleet driver: it makes “interrupt worker 3” land on worker 3's current incarnation rather than being dropped, duplicated, or delivered to a replacement.

### Worker fleet

Workers remain full harness sessions with their own tools, context management, approval behavior, and isolated workspaces. Baton does not replace those capabilities; it makes them addressable and composable.

## Representative campaign

A server/client compatibility repair illustrates the full product better than a feature inventory:

1. The orchestrator declares the outcome and approves a plan.
2. Baton starts a server-focused worker and a client-focused worker in isolated worktrees, each with an exact brief and scope.
3. The orchestrator watches both. One worker asks a schema question and becomes explicitly blocked; the orchestrator answers it without stopping the other.
4. The client worker heads toward a workaround that would violate the shared contract. The orchestrator steers it mid-run toward the canonical protocol.
5. Both workers publish checkpoints and artifacts. A reviewer worker is attached to the immutable result range.
6. The driver harvests the useful results, independently checks the claims required by policy, redrives only the failed member if needed, and integrates the accepted work.
7. The completed workflow retains its messages, decisions, artifacts, and result briefing for later continuation.

The value is the coordinated campaign, not any one mechanism in isolation.

## Engineering decisions

### One command authority across surfaces

MCP, CLI, a resident authenticated bus, and an embedded API project the same application semantics. The CLI is not a second controller with subtly different lifecycle behavior. Agent and human operators can enter through different surfaces without splitting fleet truth.

### Turn-aware control rather than process-only control

Where a harness exposes meaningful turns or checkpoints, Baton can wait, nudge, claim, or resume at those boundaries instead of treating every intervention as process termination. This preserves useful session state and makes long-lived workers practical.

### Liveness is not silence detection

A slow in-flight turn may be productive. The fleet view uses admissible progress and attention evidence rather than declaring any quiet worker dead. Escalation can move from message or checkpoint intervention to preserve-first termination, with the current worker identity and recovery state retained.

### Results are artifacts, not transcript summaries

Source changes live in git-backed artifacts; run state and communication remain in the coordination substrate. This keeps the orchestrator's context from becoming a concatenation of worker transcripts and makes results independently inspectable.

### The fast coordination spine and slow knowledge layer remain distinct

Live events, worker attention, and task claims must operate at fleet tempo. Cross-run findings, routing evidence, contradictions, and reusable knowledge require slower promotion and stronger provenance. Baton supports both without forcing every event into a heavyweight knowledge graph or treating the event tail as permanent project understanding.

## Status

The current implementation includes the run-centric fleet application, multi-worker waves, declarative workflows, heterogeneous harness adapters, shared command surfaces, live interaction and attention state, mid-run control paths, worktree isolation, result materialization, recovery, a durable coordinator, and verification/integration support.

The active frontier is deeper orchestration rather than a change of product identity: worker-led sub-fleets, richer mid-flight workflow mutation, tighter shared cells and knowledge promotion, broader native adapter parity, stronger wake-up/notification behavior, and more complete control-surface conformance.

## Deliberate limits

- Baton does not reduce full harnesses to raw completion APIs.
- It does not promise identical native control across vendors; capability cards expose the differences.
- It does not make unrestricted worker-to-worker chat the primary coordination mechanism.
- It does not turn every event or worker statement into project knowledge.
- It does not treat successful process exit as sufficient result harvest.
- It does not replace a long-horizon research evidence system such as Project Manager.
- Distributed and remote operation can extend the architecture, but the current product is centered on an owner-controlled fleet rather than general agent federation.

## Relationship to the other systems

- **HomeCloud** can supply local models, sandboxes, and worker capacity. Baton remains responsible for fleet membership, communication, steering, coordination, and harvest.
- **Project Manager** can receive selected findings and decisions after a campaign. It does not supervise Baton's live workers.
- **Flip** can be a repository or product that Baton helps develop, but Baton does not own Flip's users, content, permissions, or AI-participant semantics.

## Source

The implementation and deeper engineering record are available in the public [Baton repository](https://github.com/wahargis/baton). The portfolio case study presents the stable product and architecture rather than reproducing its campaign history and issue inventory.
