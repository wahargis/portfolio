# Portfolio architecture

The portfolio covers four distinct engineering planes. The useful organizing principle is not “four AI projects”; it is **four forms of authority that an AI-enabled system needs**:

1. a product authority over users, content, permissions, and durable experience;
2. a knowledge authority over evidence, beliefs, decisions, and research continuity;
3. a control authority over agent lifecycle, resource ownership, verification, and adoption;
4. a compute authority over model capacity, execution isolation, scheduling, and recovery.

## Responsibility map

| Concern | Flip | Project Manager | Baton | HomeCloud |
|---|---:|---:|---:|---:|
| Human-facing collaboration product | **Owns** | Does not own | Does not own | Does not own |
| Chat, forum, identity, moderation, product permissions | **Owns** | Does not own | Does not own | Does not own |
| AI participation inside a product context | **Owns** | Supports context | Controls external workers | Hosts models/execution |
| Research evidence and belief revision | Links sources | **Owns** | Emits execution evidence | Produces candidate evidence |
| Project phases and computed next work | Does not own | **Owns** | Executes declared work | Supplies capacity |
| Full coding-agent process lifecycle | Does not own | Does not own | **Owns** | May host processes |
| Worktree isolation and source adoption | Does not own | Records decisions | **Owns** | Supplies sandboxes |
| Model serving and GPU allocation | Consumes endpoints | Consumes endpoints | Selects routes | **Owns** |
| Agent sandbox and runtime checkpointing | Product-level jobs | Session records | Worker lifecycle | **Owns** |
| Verification of code changes before adoption | Product CI | Records findings | **Owns policy** | Supplies evaluators |

“Owns” means the system defines the canonical state and invariants for that concern. A neighboring system may expose data or capacity but does not redefine those semantics.

## Composition model

```text
Human communities
       |
       v
     Flip  <---------------------->  model endpoints
       |                                  ^
       | product artifacts                |
       v                                  |
Project Manager <---- durable handoff ----+---- HomeCloud
       ^                                  |
       | findings / decisions             | execution capacity
       |                                  v
       +------------------------------- Baton
                         verified agent work
```

The arrows are integration contracts, not evidence that every deployment must wire all four systems together.

### Flip and HomeCloud

Flip uses provider-compatible inference boundaries. A local HomeCloud endpoint can therefore replace or supplement a hosted provider without changing the product’s chat, forum, permissions, synthesis, or provenance model.

The contract is deliberately asymmetric:

- Flip decides the product context, tool catalog, actor scope, response persistence, and user-visible behavior.
- HomeCloud decides which model instance receives the request, how capacity is scheduled, and how local inference is supervised.
- Neither system should infer the other’s authority from a model response.

### Baton and HomeCloud

Baton treats a worker as a controllable full-session harness with declared capabilities and lifecycle semantics. HomeCloud can supply local models, sandboxes, and GPU capacity, but Baton remains responsible for:

- exact worker route and scope;
- launch identity and resource ownership;
- command/event ordering;
- steering and interaction state;
- result capture;
- independent verification;
- adoption or integration policy.

This prevents compute availability from becoming implicit control authority.

### Baton and Project Manager

Baton’s event log answers “what happened during this run?” Project Manager’s evidence graph answers “what should the project continue to believe, why, and what follows?”

A durable handoff can promote selected run outputs into:

- experiments and findings;
- hypotheses or contradiction edges;
- causal decisions;
- constraints;
- phase progress;
- session summaries.

The boundary matters. Raw worker telemetry is not automatically institutional knowledge, and Project Manager does not become a process supervisor.

### Flip and Project Manager

Flip preserves product-native provenance: source messages, threads, replies, citations, artifacts, and user feedback. Project Manager can hold a more explicit research model when a community workflow needs hypotheses, experiments, decisions, or constraints. It does not replace the social product’s own content model.

## Cross-cutting architecture principles

### 1. Bounded agency

A model sees only the tools and state relevant to the current actor, surface, and task. The code path—not the prompt—owns the final allowlist and effect boundary.

### 2. Closed lifecycle vocabularies

Important state transitions use explicit enums, typed records, or constrained schemas. Free-form narration can explain a transition; it cannot substitute for one.

### 3. One authority, several projections

Where a system offers CLI, MCP, web, native, or embedded interfaces, those interfaces project one domain model. A new surface must not quietly create a second interpretation of commands, status, or permissions.

### 4. Durable evidence over self-report

Workers and models may report completion, confidence, or findings. Trust is established through preserved source links, database constraints, independent checks, fresh environments, deterministic evaluators, or explicit human review.

### 5. Isolation before autonomy

Long-running work is given owned processes, worktrees, containers, slots, checkpoints, and cleanup obligations before it is given wider autonomy.

### 6. Local-first without local-only coupling

HomeCloud demonstrates self-hosted inference and controlled local execution. The other systems retain provider-compatible boundaries so the architecture does not depend on one machine, model family, or deployment topology.

### 7. Research modules do not masquerade as platform guarantees

Experimental search, optimization, and evaluation techniques are labeled separately from the runtime capabilities they exercise. A research workload may be technically substantial without being part of the production contract.

## Supporting systems

### Flip client

The native/web client layer belongs to Flip’s product architecture. Its concerns include React/TypeScript presentation, real-time synchronization, desktop packaging, mobile packaging, optimistic state, offline behavior, notifications, and endpoint selection. It is documented under Flip rather than promoted as a separate flagship.

### HomeCloud tools

Dispatch adapters, assistant plugins, and collaboration tooling connect external harnesses to HomeCloud and informed the later Baton control-plane work. They are supporting integration surfaces, not a fifth platform plane.

## Public documentation contract

The portfolio should remain detailed about:

- product aims and user-visible behavior;
- component and data boundaries;
- state models and invariants;
- execution lifecycles;
- trust, recovery, and failure semantics;
- implemented versus experimental capability;
- meaningful architectural tradeoffs.

It should remain restrained about:

- private source structure where disclosure adds no architectural value;
- credentials, host paths, private data, and exact operational thresholds;
- internal campaign history, issue-number inventories, and implementation diaries;
- self-authored novelty claims or metrics without a reproducible evidence package;
- demo reset procedures and reviewer choreography repeated across multiple pages.
