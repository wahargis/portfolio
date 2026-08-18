# Portfolio architecture

The portfolio covers four distinct engineering planes. The organizing principle is not “four AI projects”; it is four different kinds of system responsibility:

1. **product authority** over users, content, permissions, authorship, and durable experience;
2. **knowledge authority** over evidence, beliefs, decisions, and research continuity;
3. **fleet-orchestration authority** over delegation, live coordination, steering, and handoff among full coding harnesses;
4. **compute authority** over model capacity, execution isolation, scheduling, and recovery.

## Responsibility map

| Concern | Flip | Project Manager | Baton | HomeCloud |
|---|---:|---:|---:|---:|
| Human-facing collaboration product | **Owns** | Does not own | Does not own | Does not own |
| Chat, forum, identity, moderation, product permissions | **Owns** | Does not own | Does not own | Does not own |
| AI participation inside a product context | **Owns** | Supports context | Does not own | Hosts inference |
| Research evidence and belief revision | Links sources | **Owns** | Produces candidate run evidence | Produces candidate experiment evidence |
| Project phases and computed next work | Does not own | **Owns** | Executes delegated work | Supplies capacity |
| Cross-harness delegation and parallel workflows | Does not own | Does not own | **Owns** | May host workers |
| Worker messaging, attention, telemetry, and steering | Does not own | Does not own | **Owns** | Exposes runtime signals |
| Full coding-harness session lifecycle | Does not own | Does not own | **Owns** | May host processes/sandboxes |
| Shared in-run coordination and result harvest | Does not own | Long-horizon graph | **Owns** | Supplies storage/execution primitives |
| Model serving and GPU allocation | Consumes endpoints | Consumes endpoints | Selects worker routes | **Owns** |
| Agent sandbox and runtime checkpointing | Product-level jobs | Session records | Worker/run recovery | **Owns execution environment** |
| Verification of code changes | Product CI | Records findings | Supports fleet decisions and integration | Supplies evaluators/capacity |

“Owns” means the system defines the canonical state and invariants for that concern. A neighboring system may supply capacity or receive a handoff without redefining those semantics.

## Composition model

```text
Human communities
       |
       v
     Flip  <---------------------->  model endpoints
       |                                  ^
       | product artifacts                |
       v                                  |
Project Manager <---- selected handoff ---+---- HomeCloud
       ^                                  |
       | findings / decisions             | worker + inference capacity
       |                                  v
       +------------------------------- Baton
                    orchestrated harness work
```

The arrows are optional integration contracts. They do not imply that every deployment wires all four systems together.

### Flip and HomeCloud

Flip uses a provider-compatible inference boundary. A HomeCloud endpoint can replace or supplement a hosted provider without changing Flip’s chat, forum, permission, curation, provenance, or artifact model.

The boundary is asymmetric:

- Flip decides product context, admitted tools, actor scope, response persistence, and user-visible behavior.
- HomeCloud decides which healthy local endpoint and GPU capacity serve the request.
- Neither system derives authority from a model response.

### Baton and HomeCloud

Baton’s unit of delegation is a **full coding harness session**, not a raw model completion. HomeCloud can supply local inference, sandboxes, and execution capacity, but Baton remains the fleet driver:

- it chooses and starts worker seats;
- sends briefs and follow-up messages;
- observes liveness and progress;
- answers blocking questions;
- interrupts or steers workers mid-run;
- coordinates parallel waves and declared workflows;
- preserves shared run state and result artifacts;
- decides how work is harvested, reviewed, or integrated.

HomeCloud makes workers and models available. Baton makes them operate as a directed fleet.

### Baton and Project Manager

Baton coordinates work at run and workflow tempo. Its event ledger, task state, shared notes, worker messages, and artifacts answer: **what is the fleet doing, what is blocked, and what did it produce?**

Project Manager operates at project and research tempo. Its evidence graph answers: **what should the project continue to believe, why, and what follows?**

A selected Baton handoff can become:

- an experiment and findings;
- a supported or contradicted hypothesis;
- a decision and its rationale;
- a constraint;
- phase progress;
- a session summary.

Raw worker chatter and telemetry are not automatically institutional knowledge, and Project Manager does not become Baton's worker supervisor.

### Flip and Project Manager

Flip preserves product-native provenance: source messages, threads, replies, citations, artifacts, and participant feedback. Project Manager can hold a more explicit research model when a community workflow needs hypotheses, experiments, decisions, or constraints. It does not replace Flip’s social content or authorization model.

## Cross-cutting architecture principles

### 1. Model judgment is bounded by system-owned authority

A model may plan, interpret, synthesize, route, or intervene. Code and durable state own permissions, lifecycle, concurrency, resource identity, and irreversible effects.

### 2. Closed lifecycle vocabularies

Important state transitions use explicit states, typed records, or constrained schemas. Free-form narration can explain a transition; it cannot substitute for one.

### 3. One authority, several projections

Where a system offers CLI, MCP, web, native, resident, or embedded interfaces, those interfaces project one domain model. A new surface must not quietly create a second interpretation of commands, state, or permissions.

### 4. Preserve evidence at the boundary where it matters

Flip preserves source and authorship relationships. Project Manager preserves epistemic causality. Baton preserves run, interaction, artifact, and coordination evidence. HomeCloud preserves infrastructure and execution evidence.

### 5. Isolation before autonomy

Long-running work receives owned sessions, worktrees, containers, slots, checkpoints, and cleanup obligations before it receives wider autonomy.

### 6. Full harnesses remain first-class

Baton does not erase vendor harness behavior behind a lowest-common-denominator model API. Harness-native tools, context management, sessions, approvals, and control surfaces are capabilities to negotiate and compose.

### 7. Local-first does not mean local-only coupling

HomeCloud demonstrates self-hosted inference and execution. The other systems retain compatible boundaries so the architecture does not depend on one machine, model family, or deployment topology.

### 8. Research modules do not masquerade as platform guarantees

Experimental optimization and evaluation techniques are labeled separately from the operating contracts they exercise. A technically substantial research workload is not automatically a production guarantee.

## Supporting systems

### Flip client

The web/native client layer belongs to Flip’s product architecture. It carries synchronization, optimistic interaction, offline semantics, desktop/mobile packaging, notifications, and endpoint configuration. It is not a separate flagship system.

### HomeCloud tools

Dispatch adapters, assistant plugins, and collaboration tooling connect external harnesses to HomeCloud and informed Baton's cross-harness control work. They are supporting integration surfaces rather than a fifth architecture plane.

## Public documentation contract

The portfolio should remain detailed about:

- the problem and user outcome each system exists to address;
- architecture and authority boundaries;
- state models and invariants;
- representative execution and failure lifecycles;
- trust, recovery, and integration semantics;
- implemented versus experimental capability;
- meaningful tradeoffs and rejected alternatives.

It should remain restrained about:

- private source structure where it adds no architectural understanding;
- credentials, host paths, private data, and security-sensitive thresholds;
- internal campaign history, issue-number inventories, and implementation diaries;
- exhaustive feature/module/type catalogs without explanatory value;
- self-authored novelty or metric claims without a reproducible evidence package;
- repeated demo reset instructions and reviewer choreography.
