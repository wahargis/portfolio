# Baton

**A fleet driver for planning, routing, supervising, and completing work across persistent coding-agent harnesses.**

A modern coding harness is more than a model endpoint. Claude Code, Codex, Kimi, GLM-backed tools, Grok-oriented clients, and similar systems each bring their own file and shell tools, permission model, context management, session behavior, model controls, and vendor-specific strengths. Running several of them side by side does not create a coordinated engineering team; it creates several isolated applications that a person or lead agent must manually prompt, watch, reconcile, and clean up.

Baton adds the missing fleet layer. One orchestrator—or a human using the same application surface—states an outcome, approves a plan, chooses or constrains the exact harness/model/effort routes, and follows one durable **Run** while Baton launches persistent workers, carries questions and decisions, exposes attention, steers active sessions, coordinates parallel work, verifies candidate results, and closes the resources it owns.

The product is the **fleet driver**. Worktrees, event records, process supervision, route attestations, and verification gates matter because they make that driving dependable. They are supporting mechanisms, not a substitute description of what Baton is for.

![Baton fleet-driver architecture](assets/fleet-driver.svg)

## At a glance

| Concern | Baton’s answer |
|---|---|
| **Who it is for** | A lead coding agent or human supervising substantial repository work that benefits from several persistent, differently capable coding harnesses. |
| **What the operator sees** | One Run with an objective, approved plan, worker roster, exact routes, progress, questions, blocked decisions, candidate results, verification, and cleanup state. |
| **What a worker remains** | A full vendor-native coding session with its own tools, context, permissions, and resumability—not a stateless raw-model completion. |
| **What Baton owns** | Fleet planning and dispatch, worker/session identity, route resolution, communication, attention, steering, repository isolation, evidence, recovery, result handling, and resource lifecycle. |
| **Primary implementation shape** | A Node.js application and coordinator with harness adapters, repository worktrees, durable coordination state, authenticated Web and MCP northbound surfaces, CLI access, and embedded APIs. |
| **Source boundary** | Development remains private. Selective public release is planned at [flip-engineering/baton](https://github.com/flip-engineering/baton); that organization repository has not yet been published. |

## What a Baton Run looks like

Suppose a repository has a compatibility defect spanning a server contract, a native client, and integration tests. A single agent could work serially, but the task has useful parallel structure and different reasoning needs.

A Baton Run can proceed as follows:

1. The orchestrator states the outcome and definition of done rather than issuing several disconnected worker prompts.
2. Baton materializes an inspectable Plan: one member analyzes the server boundary, another traces the client projection, and a third prepares independent verification or review.
3. Each member receives an explicit repository scope and route. The orchestrator can pin Codex for one role, a Claude-family harness for another, or constrain model and effort while letting route policy choose among admitted options.
4. Baton creates the required worktree/session resources and launches persistent harnesses rather than fire-and-forget subprocesses.
5. The RunView shows which members are ready, active, waiting for input, blocked by dependencies, complete, or failed. It also shows which route actually served each member.
6. A worker can ask a decision question. The request becomes Run attention, receives one authoritative answer, and returns to the correct session instead of disappearing in terminal output.
7. The orchestrator can send a normal message, nudge work at a turn boundary, redirect a member, interrupt it, or stop it. Baton distinguishes a requested stop from a confirmed stop.
8. Candidate changes and findings are harvested by identity. Declared checks can be rerun by Baton in a clean checkout rather than accepted solely because a worker says the task is complete.
9. The orchestrator reviews or selects the result, and Baton records what was adopted or integrated separately from what a worker merely proposed.
10. The Run closes owned sessions, worktrees, reservations, and other resources under explicit lifecycle rules.

The operator follows one evolving application object. They do not have to reconstruct the project from several chat windows, process IDs, branches, and hand-written status notes.

## Why full harnesses are the worker unit

Baton deliberately preserves vendor-native coding environments instead of reducing every worker to a generic `prompt -> model -> text` API.

A full harness may provide:

- repository-aware file, shell, git, browser, and search tools;
- vendor-tuned system behavior and tool protocol;
- native planning, retries, approvals, and permission policy;
- model and reasoning-effort controls;
- context compaction and session continuation;
- resumable, forkable, or attachable execution;
- subscription-backed capacity that is not equivalent to an API call.

Those behaviors materially affect engineering performance. “A model family through one harness” and “the same or related model through another harness” are different execution routes because the surrounding tools and session semantics differ.

Baton therefore normalizes the **meaning of fleet controls** without pretending the underlying harnesses are identical. Adapter capability cards can report a control as native, emulated, or unsupported. The application can route accordingly and remain honest about what occurred.

## The product model: one Goal, one Plan, one Run

The ordinary Baton surface is run-centric:

```text
outcome + constraints
  -> inspectable Goal and Plan
  -> approval
  -> dependency-aware worker dispatch
  -> one evolving RunView
  -> messages, questions, steering, and attention
  -> candidate results and coordinator-observed evidence
  -> review / adoption / integration
  -> confirmed cleanup
```

Low-level worker commands still exist for embedding, debugging, and emergency control. They are not the workflow the ordinary user should have to assemble manually.

### Goal and Plan

The application turns a concise outcome into explicit operating state:

- objective and success conditions;
- members or task nodes, scopes, and dependencies;
- requested or constrained harness/model/effort routes;
- expected artifacts and result contracts;
- decisions that require orchestrator or human approval;
- verification and review requirements;
- lifecycle and cleanup ownership.

The Plan is data that can be inspected, approved, revised, replayed, and related to the eventual result. It is not merely an early paragraph in a worker transcript.

### RunView

The RunView is the bounded representation of the assembled fleet. It answers the questions an operator actually has:

- What is the Run trying to accomplish?
- Which plan nodes are available, active, blocked, complete, or failed?
- Which worker/session owns each piece of work?
- Which exact route was requested, selected, and observed?
- What needs attention now?
- Which questions, findings, artifacts, or candidate results exist?
- Which verification, review, adoption, or integration step remains?
- Which resources still belong to the Run?

Detail is progressively disclosed. An operator can orient from the high-level Run state and then inspect one member, episode, workstream, evidence record, or lifecycle event when necessary.

## Architecture

### 1. Orchestrator

The orchestrator can be a capable coding harness or a human using the CLI or Web surface. It retains semantic authority over the work: the desired outcome, decomposition, route preferences, plan approval, answers to ambiguous questions, redirection, result selection, and any higher-authority publication decision.

Baton does not try to replace that judgment with a generic autonomous scheduler.

### 2. Run application

The application is the product-facing layer. It compiles or accepts Goals and Plans, admits and starts Runs, schedules dependency-ready members, presents Run and workstream state, handles attention and operator actions, interprets workflows, and coordinates harvest, review, adoption, and integration.

The implementation exposes a common command registry for operations such as starting and inspecting Runs, following progress, approving plans, answering questions, messaging or stopping workstreams, recording feedback, inspecting evidence, and adopting results. Web and MCP transports project that same application rather than inventing unrelated orchestration vocabularies.

### 3. Fleet kernel

The coordinator owns the mechanical state that should not depend on an LLM remembering every race:

- worker and session incarnation identity;
- launch, attach, resume, interrupt, stop, and reap transitions;
- repository worktrees and path/capacity ownership;
- commands, events, cursors, and recoverable projections;
- route and worker-policy resolution;
- result and verification identities;
- restart recovery, draining, and cleanup.

This kernel is deliberately plain code. A delayed message should not reach a replacement worker; a process should not be reported stopped merely because a signal was sent; and a restart should not require an agent to infer fleet state from incomplete logs.

### 4. Harness adapters and persistent workers

Southbound adapters map Baton's lifecycle and communication semantics onto each harness’s native control surface. The implementation includes adapter and session layers for several coding environments rather than requiring every provider to expose the same protocol.

Workers retain their native tools, context, and turn behavior. Source-modifying members receive isolated repository contexts. They can continue over several turns, receive follow-up instructions, survive orchestrator compaction through a bounded resume briefing, or be selectively replaced without restarting the whole Run.

## Route control is explicit

Baton models three independent route axes:

- **harness** — the actual coding environment;
- **model** — the exact model requested or resolved inside that environment;
- **effort or service tier** — the harness-native reasoning or service control where available.

The orchestrator can pin an axis, allow a set of routes, deny unsuitable options, or let policy choose among live capability cards. Requested, resolved, and provider-observed route identity remains inspectable.

Silent substitution is not treated as harmless. A worker running through an unrequested harness or default model can have different tools, permissions, context behavior, cost, and quality. Route divergence is therefore evidence the Run must expose, not a detail to hide behind a generic worker name.

## Communication, attention, and steering

Baton separates cooperative communication from preemptive control.

### Communication

Ordinary messages, worker questions, orchestrator answers, plan proposals, progress notes, and result briefings can respect the harness’s turn boundaries. A blocking question becomes an explicit Run attention item. Its answer is tied to that request and delivered to the intended member.

This prevents an orchestrator from having to poll every terminal or infer that silence means a worker needs help.

### Steering

A running session may need more than a message. Baton can represent checkpoint nudges, redirects, interruption, stop, and cleanup as distinct controls. The adapter reports whether the harness provides native support or whether Baton must emulate the behavior.

Confirmed lifecycle is important. The system distinguishes:

```text
stop requested -> stopping -> worker/process confirms closure -> stopped/reaped
```

That two-phase model prevents the Run from reallocating or deleting resources while the prior worker may still be writing to them.

### Attention

The RunView distinguishes normal progress from states that require judgment: a worker waiting for an answer, a deferred human decision, a suspected stall, a route/capacity refusal, a candidate result awaiting review, or a member that can be selectively redriven.

Attention is a product-level summary of what the fleet needs, not a raw telemetry stream.

## Parallel work and workflow-as-data

A **wave** groups several members under one durable execution identity. Members can use different routes, scopes, roles, and result contracts. Baton can start or attach to the wave, inspect its roster and progress, communicate with selected members, preserve their result identities, and redrive eligible failures without discarding successful work.

A declarative workflow describes the coordination pattern itself: members, dependencies, routes, scopes, steering policy, decision points, and harvest conditions. The workflow interpreter owns launch and lifecycle rather than forcing every caller to write a bespoke loop around primitive worker commands.

This supports common engineering patterns:

- parallel server and client analysis followed by an integration member;
- independent implementation candidates followed by review and selection;
- one design member feeding several bounded implementation tasks;
- a research or repository-orientation phase that produces handles consumed by later workers;
- selective redrive of failed work while preserving verified results.

Deeper worker-orchestrated hierarchies and more dynamic workflow mutation remain active development areas. They are not required to understand or use the run-centric core.

## Shared context without transcript flooding

A fleet needs more than worker-to-worker chat. Baton maintains addressable shared state for current plans, artifacts, findings, result handles, repository representations, short-lived working notes, and selected longer-lived knowledge.

The design follows three rules:

1. **Push the minimum; pull detail by handle.** Workers receive the authoritative brief and high-signal current facts. Larger artifacts or peer results remain addressable without being copied into every prompt.
2. **Preserve provenance.** Operator constraints, coordinator-computed state, worker-authored conclusions, and references to external artifacts remain distinguishable.
3. **Promote deliberately.** A scratch observation from one worker does not automatically become project truth. Useful results can be selected, verified, or promoted at an appropriate horizon.

This keeps the lead context usable while still allowing workers to share evidence and build on prior results.

## Verification supports orchestration; it is not the product

Workers are expected to be capable, but their completion claims are still candidate evidence. Baton can rerun declared checks in a clean repository state, compare the observed result with the worker’s claim, request independent review, preserve several candidates, and separate result adoption from source integration.

The private implementation includes a deterministic demonstration against the assembled driver:

- an honest worker creates the requested change, and the coordinator independently reruns the declared check before accepting completion;
- a worker claims success without producing the required result, and the same clean-checkout verification catches the divergence and records failure;
- an interrupt enters `stopping` immediately but does not become `stopped` until the worker confirms, preventing a delayed edit from landing after control was supposedly revoked.

Only the workers in that demonstration are mocked. The git isolation, coordinator lifecycle, stop semantics, verification, and route learning are the real assembled code paths.

These controls matter because a fleet driver needs to know whether to continue, redrive, compare, review, or integrate. They remain stages inside the larger Run lifecycle.

## Result, review, adoption, and integration remain distinct

Baton does not collapse every positive worker outcome into an automatic merge:

- a **worker result** is the harness’s candidate output;
- **verification** is coordinator-observed executable evidence;
- **review** is an independent semantic assessment;
- **adoption** selects and preserves a result without necessarily changing the target source;
- **integration** applies an allowed source effect under policy;
- publication or deployment requires separate authority.

Keeping those states separate makes partial success, comparison, rollback, and human approval possible without losing the evidence that produced the decision.

## One application across embedded, CLI, Web, and MCP

Baton exposes the same Run-oriented application through:

- direct embedding for another orchestrator or service;
- the `baton` CLI;
- an authenticated resident/Web command surface;
- MCP stdio and a Web/MCP bridge.

The implementation is organized around one semantic command registry so these surfaces can share Run identity, operations, capability requirements, argument validation, evidence, and refusal behavior. A new transport should not become a weaker parallel fleet implementation.

## Implementation status

### Implemented and exercised in the private development repository

The current codebase includes the assembled driver and Run application, persistent adapter-backed workers, exact route tuples, worktree and process ownership, messages and questions, attention, interruption and confirmed stopping, restart-oriented coordination state, waves and declarative workflows, Web/CLI/MCP/embedded surfaces, clean-checkout verification, review/adoption/integration stages, and result/resource cleanup.

The codebase is broader than this case study, but the portfolio intentionally documents stable operator-facing capabilities rather than reproducing every internal module, event kind, schema ceiling, or issue chronology.

### Active development

Current work includes stronger end-to-end parity and actionability across surfaces, richer harvest and redrive continuity, broader native harness control, worker-facing review feedback, deeper nested orchestration, and more complete shared code/repository representations.

### Longer-horizon work

More dynamic program/workflow representations, multi-machine control, advanced semantic integration, broader computer-use capability, and systematic fleet learning remain research or future product directions rather than baseline claims.

## Limitations

- Vendor harnesses expose materially different controls; some operations must be emulated or refused rather than normalized perfectly.
- Persistent sessions and parallel work consume substantial context, subscription, process, and repository capacity; route and worktree limits remain real.
- Independent verification can establish that declared checks passed, not that the implementation is globally correct or the right product decision.
- Shared summaries and briefings can omit or distort useful context; addressable source artifacts reduce but do not eliminate that risk.
- A durable coordinator can recover its own state, but it cannot guarantee that every external harness supports equivalent session recovery.
- More workers do not automatically improve a task. Decomposition, scopes, dependencies, and result contracts still require good judgment.
- Automatic source integration remains intentionally more constrained than worker execution.

## Non-goals

Baton is not:

- a raw-API multi-agent framework that discards vendor harness behavior;
- a subprocess wrapper that treats stdout and process exit as sufficient lifecycle;
- a verification product with orchestration attached as an afterthought;
- a lowest-common-denominator adapter that hides native route differences;
- an unrestricted worker-chat mesh;
- a replacement for the domain knowledge of Project Manager or the infrastructure scheduling of HomeCloud;
- a collection of primitives that requires every caller to rebuild the Run application manually.

## Relationship to the other systems

- Baton can operate on Flip or any other repository, but it does not own that product’s users, content, or permissions.
- Baton can produce findings, decisions, candidate changes, and run summaries that may later be promoted into Project Manager; Project Manager does not supervise Baton's live sessions.
- HomeCloud can supply local model or sandbox capacity, while Baton retains harness route, worker, communication, Run, and result authority.

## Source

Selective public release is planned at [flip-engineering/baton](https://github.com/flip-engineering/baton). That repository does not yet exist publicly. This portfolio intentionally does not link to the private personal development repository.