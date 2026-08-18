# Baton

**Cross-harness fleet orchestration for full-session coding agents.**

Baton lets one orchestrator agent—or a human using the same surface—direct a persistent fleet of full coding harnesses across vendors. A caller states an outcome, chooses or constrains the harness/model/effort route, reviews a generated Plan, and follows one Run as Baton dispatches work, carries questions and decisions, steers active sessions, coordinates parallel members, preserves shared context, harvests results, and cleans up.

The product is the **fleet driver**. The reliable coordinator, event ledger, worktrees, and verification gates are supporting systems that make the driving usable; they are not Baton's identity.

![Baton fleet-driver architecture](assets/fleet-driver.svg)

## Why full harnesses are the unit of work

Baton deliberately orchestrates Claude Code, Codex, GLM-through-Claude, Grok Build, DeepSeek, Kimi, and other full-session harnesses rather than reducing every worker to a raw model API call.

A harness already contains a substantial execution environment:

- vendor-tuned system behavior and prompting;
- native file, shell, git, browser, search, and approval tools;
- sandbox and permission policy;
- context management and compaction;
- retries, planning, and turn semantics;
- resumable or forkable sessions;
- subscription-backed seats and vendor-specific model controls.

The harness is therefore part of the model's effective capability. Baton preserves those strengths and adds the fleet layer the individual products do not provide: heterogeneous routing, persistent multi-worker execution, shared state, cross-harness communication, steering, workflow composition, and fleet-wide capabilities.

## Product model: one Run, not a bag of worker commands

The ordinary Baton interface is run-centric.

```text
concise outcome + route/profile constraints
  -> proposed Goal and Plan
  -> explicit approval
  -> dependency-aware dispatch
  -> one evolving RunView
  -> questions, messages, steering, and progress
  -> result harvest and optional review
  -> evidence, adoption/integration, and cleanup
```

The user should not have to manually reproduce orchestration from low-level `spawn`, `send`, `interrupt`, `result`, and `kill` calls. Those remain useful kernel and emergency controls, but the application owns normal choreography.

### Goal and Plan

The Run application turns a concise outcome into a bounded plan with:

- explicit objective and success conditions;
- selected harness/model/effort routes;
- members or task nodes;
- dependencies and scopes;
- steering and decision policy;
- expected artifacts and harvest conditions;
- verification/review requirements;
- lifecycle and cleanup ownership.

The Plan is inspectable and approved before consequential worker effects. Once approved, Baton lowers it into the coordinator's lifecycle primitives.

### RunView

A RunView is the bounded, progressively disclosed representation of the fleet state. It answers:

- What is the Run trying to accomplish?
- Which Plan nodes are ready, active, blocked, complete, or failed?
- Which harness/model/effort actually serves each member?
- What needs orchestrator or human attention?
- What shared artifacts, findings, or result pins exist?
- Which verification/review/adoption steps remain?
- Which resources still belong to the Run?

The caller follows one application object rather than collecting unrelated receipts from several surfaces.

## Architecture

### 1. Orchestrator layer

The orchestrator remains an AI harness such as Claude Code or Codex, or a human operating through CLI/Web. It decides:

- the outcome;
- decomposition and route preferences;
- plan approval and amendments;
- answers to worker questions;
- when to nudge, redirect, review, redrive, or stop;
- which verified result to adopt or integrate.

Baton does not replace the orchestrator's semantic judgment.

### 2. Run application

The application compiles Goals and Plans, schedules dependency-ready work, materializes one RunView, interprets declarative workflows, manages attention, and coordinates harvest/review/adoption.

This is the product layer ordinary callers use.

### 3. Fleet kernel

A plain-code coordinator carries out the mechanical work that should not depend on an LLM remembering every race and resource:

- launch and attach to full harness sessions;
- carry commands and events durably;
- preserve worker/session identity;
- own worktrees and process resources;
- enforce route and capacity constraints;
- confirm lifecycle transitions;
- rebuild projections after restart;
- recover, stop, and reap exact resources.

The kernel makes orchestration dependable; it does not define Baton's purpose.

### 4. Southbound adapters

Each harness adapter maps Baton's common semantics onto native capabilities:

- start/resume/fork/attach;
- model and effort selection;
- message and turn APIs;
- questions and approvals;
- native or emulated steering;
- usage/context events;
- interruption and close;
- result/session capture.

Adapter cards describe controls as **native**, **emulated**, or **unsupported**. Baton unifies the meaning of a control without pretending every harness implements it identically.

### 5. Persistent full-session workers

Workers keep their vendor-native tools, context, and session state. Each source-modifying member receives a separate repository/worktree context and an exact route. Baton can resume a worker, continue with follow-up turns, or replace/redrive selected members without flattening the fleet into stateless completions.

## Harness, model, and effort are independent route axes

Baton treats these as separate choices:

- **harness:** Claude Code, Codex, Grok Build, native Kimi, or another adapter;
- **exact model:** a specific model exposed by that harness;
- **effort/service tier:** reasoning effort or equivalent native control.

The orchestrator can pin any axis, constrain allowed/denied routes, or let policy choose among live capability cards. Requested, resolved, and provider-observed identities remain visible in Run state and evidence. Silent fallback to an unrelated harness or default model is a contract failure.

This matters because “Claude harness using Kimi” and “native Kimi harness” are different execution environments even if they reach a related model family.

## Parallel work: waves, workflows, and hierarchy

### Waves

A wave groups several members under one durable identity. Members can have different routes, scopes, roles, and result contracts. Baton can:

- start members together;
- attach to an existing wave;
- inspect roster, phase, progress, and attention;
- message or steer selected members;
- preserve each member's result as an immutable handle;
- redrive only eligible failed members;
- emit a wave-level briefing and harvest result.

### Workflow-as-data

A declarative workflow describes the orchestration pattern rather than requiring bespoke driver code.

```json
{
  "objective": "Audit and repair a client/server compatibility boundary",
  "members": [
    {
      "id": "server-analysis",
      "route": {"harness": "codex", "model": "exact-model", "effort": "high"},
      "scope": ["server/contracts", "server/tests"]
    },
    {
      "id": "client-analysis",
      "route": {"harness": "claude-code", "model": "exact-model", "effort": "medium"},
      "scope": ["client/sync", "client/tests"]
    }
  ],
  "steering": {
    "approveAdvertisedPlan": true,
    "askOnDecision": true,
    "nudgeOnCheckpoint": true
  },
  "harvest": {
    "require": ["findings", "candidate", "verification"]
  }
}
```

The workflow interpreter owns member launch, attention, steering policy, decision deferral, completion, and harvest. The specification is a first-class Run input, not a prompt transcript.

### Hierarchical orchestration

Baton's larger design supports a heavyweight worker coordinating cheaper or more specialized members inside one declared workflow. This allows patterns such as:

- orchestrator → coordinator worker → high-throughput implementation rows;
- one design/review member supervising several bounded tasks;
- nested decomposition with decisions escalated back to the top-level orchestrator;
- selective aggregation rather than every child flooding the lead context.

Worker-orchestrated swarms and deeper nested orchestration are active development areas, not the premise of the shipped core.

## Interaction: communication, attention, and steering

Baton separates cooperative communication from preemptive control.

### Communication channel

Used for content that can respect turn boundaries:

- authoritative brief;
- worker plan proposal;
- orchestrator answer;
- worker question;
- ordinary message/reply chain;
- progress note;
- result and briefing pack.

Blocking interactions change the Run's `waitingOn` state so the orchestrator knows action is required. Answers are tied to one request and converge on one accepted response.

### Steering channel

Used when the orchestrator changes active work:

- nudge at the next checkpoint;
- redirect the current or next turn;
- claim a paused turn;
- interrupt;
- stop or reap.

The adapter reports whether the operation is native or emulated. Turn-checkpoint steering avoids killing a productive session merely because one model turn ended.

### Human and orchestrator attention

The RunView distinguishes:

- work progressing normally;
- a worker waiting for input;
- a decision intentionally deferred to a human;
- a possible stall;
- completed members awaiting harvest/review;
- resource or route limitations.

This is more useful than a stream of raw telemetry and avoids requiring the orchestrator to poll every worker blindly.

## Context and harness engineering

Baton controls context it injects into harnesses it does not own. The design uses several rules.

### Semantics once, syntax per harness

The Goal, task brief, capability operation, and result envelope are defined in harness-neutral form, then rendered into each harness's native idiom. A Codex brief and a Claude brief can look different while expressing the same objective, scope, and definition of done.

### Push the minimum; pull the rest by handle

Workers receive the concise authoritative brief and high-signal current facts. Larger material—repo maps, source artifacts, peer results, prior findings, recordings, representations—is addressable and pulled only when needed.

This protects both worker and orchestrator context windows from transcript floods.

### Provenance-typed context

Injected context distinguishes:

- Baton-authored constraints and objectives;
- coordinator-computed facts;
- model-authored prose and summaries;
- references/handles to larger artifacts.

The distinction is used for presentation, recall, and verification; it is not a claim that workers are merely hostile inputs.

### Compaction continuity

When a harness compacts or resumes a session, Baton can re-inject the authoritative brief identity and a bounded resume digest. The goal is to preserve orchestration intent across the harness's own context lifecycle without replacing its native memory system.

## Shared coordination and memory

Baton has several memory tempos because one store cannot efficiently serve all fleet needs.

| Tempo | Purpose | Examples |
|---|---|---|
| **Operational** | What happened and what can be replayed now. | append-only events, cursors, lifecycle, telemetry, receipts. |
| **Coordinative** | What the current Run/workflow is doing. | task DAG, claims, path leases, artifact manifest, result pins, ready-work state. |
| **Scratch** | Fast live fleet working set. | scratchpads, boards, cells, memoized experiments, notices, short-lived leases. |
| **Epistemic** | Selected durable knowledge across Runs. | findings, decisions, constraints, route statistics, counterexamples, representations, contradiction/supersession. |

### Knowledge horizons

Knowledge can exist at task, workflow, and project horizons. Promotion is explicit and bounded: a worker scratch note does not automatically become project truth. Briefing and context packs expose the relevant slice for a member or the orchestrator.

### Shared substrate over unrestricted peer chat

Workers coordinate primarily through structured shared media—artifacts, boards, scratch cells, task state, code representations, and knowledge records—rather than unrestricted worker-to-worker conversation. This keeps coordination visible, addressable, and reusable.

### REPL and shared cells

The collaboration layer includes shared typed cells and cross-run scripting primitives for live coordination and memoized experiments. These are ordinary fleet objects with ownership and lifecycle, not a hidden side channel.

## Fleet-shared capability plane

Baton's complete design goes beyond controlling harnesses. It turns expensive per-worker capabilities into shared, agent-shaped services that the whole fleet can use.

| Capability family | Purpose |
|---|---|
| **Atlas** | Fleet-shared lexical/structural/symbol/code-property representations, search, code seeds, and bounded deltas across base plus worker overlays. |
| **Vantage** | Structured debugging and causal observations over DAP/record-replay assets. |
| **Evidence Ladder** | Validation from tests and mutation through property/fuzz/model-checking/proof rungs, selected by risk. |
| **Scratch / Bench** | Blackboard coordination, leases, shared facts, and memoized reproducible experiments. |
| **Skill Forge / computer use** | Verified portable skills and distillation of fragile GUI trajectories into reusable automation. |
| **Cartographer / Quartermaster** | Shared repository orientation plus dependency, reuse, SBOM, and supply-chain evidence. |
| **Cairn** | Run scorecards, causal findings/decisions, route statistics, bounded recall, contradiction, and selected cross-run knowledge. |

The common contract is more important than the names: results are structured and token-bounded, long operations are supervised, outputs are addressable, and useful evidence can be rerun or promoted. Implementation depth differs by module and is tracked in the canonical source status catalog.

## Reliability kernel: supporting the fleet driver

The coordinator carries five foundational guarantees:

1. **Incarnation/version fencing** prevents delayed commands from targeting a replacement session.
2. **Confirmed stop/reap** distinguishes requesting a stop from proving the owned process/session actually closed.
3. **Durable cursors and replay** prevent questions or events from disappearing across restart.
4. **Single-consumer interaction state** makes one request receive one authoritative answer.
5. **Append-oriented event truth** lets projections and views be rebuilt.

These mechanisms support messaging, steering, recovery, and multi-worker coordination. They belong in the architecture because they make the fleet driver work—not because Baton is fundamentally a fencing or event-log product.

## Verification, review, adoption, and integration

A Run can re-execute declared checks in a fresh worktree, add red-to-green or mutation evidence, route an independent semantic review, preserve a candidate result, and later adopt or integrate it under policy.

This matters because the driver needs an objective way to decide whether it can move on, redrive, compare candidates, or merge. It remains one stage of the larger Run lifecycle.

The stages stay distinct:

- **worker result:** candidate output from the harness;
- **verification:** coordinator-executed evidence;
- **review:** independently routed semantic assessment;
- **adoption:** select and preserve a result without integrating it;
- **integration:** apply a policy-allowed local source effect;
- **publication/deployment:** separate higher-authority actions.

## One application across embedded, CLI, Web, and MCP

Baton exposes one command registry and application semantics across:

- direct embedding;
- authenticated resident/Web bus;
- `baton` CLI;
- MCP stdio.

The surfaces should present the same Run, attention, route, workflow, evidence, and refusal semantics. Help and detail are progressively disclosed so ordinary callers see the Run application while advanced users can reach kernel controls.

## Status

| State | Capability |
|---|---|
| **Shipped core and major planes** | Run application; persistent cross-vendor workers; exact route tuples; resident/embedded/CLI/MCP surfaces; waves; workflow-as-data; turn-checkpoint steering; questions/reply lanes/attention; scratchpads, boards, briefing/context packs and knowledge horizons; REPL; fresh verification/review/adopt/integrate; code and dependency representations; durable lifecycle, recovery, capacity, drain, and reap. |
| **Active development** | Worker-orchestrated swarms; deeper nested orchestration; worker-facing verdict/coaching delivery; broader LSP support; cross-deployment knowledge; prescriptive diagnostics; tighter surface conformance and error actionability; richer harvest/redrive continuity. |
| **Longer-horizon / research** | Program IR and higher-level workflow language; dynamic wave mutation and quiescence-derived completion; broader capability-plane modules; true semantic merge; remote/multi-machine control; computer-use tiers; expanded evaluation and learning policy. |

The detailed status and evidence ledger remains in the [canonical Baton repository](https://github.com/wahargis/baton). This portfolio page intentionally removes issue-number chronology while retaining the complete architectural scope.

## Non-goals and corrections

Baton is not:

- a raw-API multi-agent framework that discards vendor harness behavior;
- a subprocess wrapper that only waits for stdout;
- a verification product or neutral “trust institution” with orchestration as an optional feature;
- a lowest-common-denominator adapter that hides native differences;
- a direct worker-chat mesh;
- a HomeCloud or Project Manager runtime integration;
- a phase-runner collection that asks callers to reconstruct the application manually.

## Relationship to the other systems

- Baton can operate on Flip or any other repository, but it does not own that product's users, content, or permissions.
- Baton can promote selected findings and decisions into Project Manager or its own project-horizon knowledge, but the two systems have different primary state models.
- Baton can consume HomeCloud-hosted endpoints or sandboxes, but it remains deployment-neutral and does not depend on one local cluster.

## Source

The full implementation, design record, capability catalog, specs, evidence, and status tiers are available in the public [Baton repository](https://github.com/wahargis/baton). The source documents that most directly define the product are:

- [`SYSTEM.md`](https://github.com/wahargis/baton/blob/master/SYSTEM.md) — authoritative complete system design;
- [`docs/19-north-star-corrected.md`](https://github.com/wahargis/baton/blob/master/docs/19-north-star-corrected.md) — explicit correction that the fleet driver is the product;
- [`docs/26-full-system-goal.md`](https://github.com/wahargis/baton/blob/master/docs/26-full-system-goal.md) — no-loss full capability goal and definition of complete.
