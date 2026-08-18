# HomeCloud

**A self-hosted AI runtime that turns local accelerators into supervised inference, recoverable agent execution, and a controlled research environment.**

Owning GPUs does not by itself produce dependable local AI. A model server can wedge, two workloads can contend for the same memory, a long agent can lose its context or sandbox, and an experimental job can degrade the interactive service that justified running the machine in the first place.

HomeCloud is the operating layer around that hardware. It represents inference as schedulable capacity, gives agent runs owned execution environments and checkpoints, keeps durable context outside any one model window, and places research workloads under the same resource and failure controls as user-facing work.

![HomeCloud architecture diagram](assets/diagrams/architecture.svg)

## The architectural problem

A self-hosted system has to answer several questions continuously:

- Which model endpoints are actually healthy and ready?
- Which request should receive scarce GPU capacity now?
- Can a request return to the same instance for prompt-cache locality?
- What happens when one GPU or model process fails while others remain healthy?
- Where does generated code execute, and who owns cleanup?
- How does a long task resume after compaction, restart, or interruption?
- How can experiments use idle capacity without becoming production authority?

HomeCloud treats those as runtime contracts rather than operator folklore.

## Separate model service lifecycle from application lifecycle

The Phoenix application can adopt model endpoints that are supervised outside its own process tree. This prevents an application restart from duplicating or orphaning GPU-bound servers and allows endpoint health, drain, and replacement to be managed independently of a chat or agent session.

The model service boundary exposes a small operational truth: endpoint identity, model/profile, context capacity, health, throughput, GPU placement, and available slots. The application does not infer that a process is usable merely because a port exists.

This separation also makes provider-compatible remote endpoints possible. Local and hosted inference can participate in the same application contract while retaining different capacity and privacy policies.

## Capacity is represented as slots, not assumptions about GPUs

The instance pool gives local inference checkout/checkin semantics. A request acquires a healthy slot that matches its profile, receives a pinned endpoint, executes, and returns the slot in a guaranteed cleanup path.

The pool can prefer an earlier instance for cache affinity, prioritize interactive work over research and background tasks, drain an endpoint before a model swap, and reclaim capacity when the caller disappears. Concurrency is derived from live slot state rather than hard-coded from the number of installed cards.

That distinction matters on heterogeneous hardware. One model may span several linked V100s; another may run as an independent replica; a modality-specific service may occupy the separate accelerator; a remote route may have an entirely different concurrency limit. HomeCloud schedules the capacity that exists instead of pretending every GPU equals one interchangeable request.

## The scheduler protects the service from its own experiments

Text inference, media jobs, CUDA work, research trials, and background maintenance can compete for the same accelerators. HomeCloud tracks demand and durable GPU claims, applies workload priority, and uses queues or exclusion locks where combinations are unsafe.

Interactive requests are not merely “another task” beside an open-ended research search. The scheduler can reserve or prefer user-facing capacity and let lower-priority work consume the remainder. An experiment therefore runs because the platform admitted it, not because it happened to start first.

## Agent execution owns an environment

HomeCloud’s execution engine supports autonomous, interactive, and plan-only work through one lifecycle. A source-modifying run receives an owned sandbox derived from project state, a persistent executor where available, bounded setup, captured tool results, cancellation state, and guaranteed cleanup.

The engine reports typed phases and events so a UI or orchestrator can distinguish reasoning, dispatch, execution, validation repair, completion, and failure. Long loops are checked for repeated actions, repeated output, regression, and stagnation. Validation failures can return to the model for bounded repair; productive work can extend through explicit stop-hook reflection rather than an arbitrary fixed turn count.

The important consequence is containment. A generated command or failed research trial runs inside an execution boundary and cannot become an implicit mutation of the Phoenix host or another agent’s workspace.

## Checkpoints make recovery different from pretending a process survived

Task state, logs, file changes, cancellation flags, and checkpoints are preserved under an explicit lifecycle. A resumed run reconstructs useful context and workspace state without claiming that the original process is still alive.

Cleanup and recovery are designed together. A retained checkpoint should not require leaking a container, and terminating a sandbox should not erase the durable evidence needed to understand or resume the task.

## Context survives beyond one model window

HomeCloud keeps several forms of context because they have different retention value:

- recent conversation supports the current turn;
- task memory carries working state across turns;
- retrieved documents provide source material selected for the task;
- learned facts and reusable skill patterns carry selected knowledge across tasks;
- a knowledge graph retains entity and relationship context;
- checkpoints and research records preserve execution evidence.

The context engine budgets and refreshes this material as work evolves. When history approaches the working limit, it compacts older context while protecting recent exchanges and critical state. A large advertised context window does not eliminate selection, provenance, or compaction error.

## A representative mixed workload

A user submits an interactive research question while a lower-priority CUDA optimization program is running.

1. The interactive request enters the execution engine and asks the inference router for a user-priority slot matching its model profile.
2. The pool selects a healthy endpoint and preserves affinity on later turns where useful.
3. The agent retrieves documents, executes bounded tools in its sandbox, and checkpoints progress.
4. The CUDA program retains its durable research state but yields or waits when its GPU claim conflicts with the user-facing service.
5. If the selected endpoint fails, the pool marks that instance unhealthy without globally blocking unrelated accelerators; the request can retry or route under policy.
6. When the user task completes, its slot and sandbox are returned, while selected facts or skills can be promoted for later work.
7. The research program resumes from its own trial and checkpoint state rather than restarting the entire platform.

This is the value of the runtime: inference, agents, and research share hardware without sharing failure or authority implicitly.

## Research is a workload, not the platform’s identity

HomeCloud hosts experimental program search, autonomous research, code evaluation, and CUDA optimization. These workloads exercise long-running scheduling, candidate generation, deterministic compilation/tests, correctness checks, profiling, and persisted trial state.

They are valuable because the platform can run and evaluate them locally. They are not universal production guarantees or evidence that every task should escalate into tree search. Reward design, search strategy, and evaluator composition remain research areas and are labeled separately from the operating runtime.

Dynamic tool synthesis is similarly high power: the runtime can register new capabilities, but deployment policy and trust boundaries must decide when generated code may become an admitted tool.

## Current hardware topology

The current lab configuration combines:

- a four-GPU NVIDIA V100 32 GB NVLink pool with 128 GB aggregate VRAM; and
- a separately attached NVIDIA A100 Drive 32 GB accelerator outside that NVLink fabric.

The architecture does not assume that these devices form one uniform tensor-parallel pool. Model placement depends on datatype/kernel compatibility, quantization, context memory, interconnect, modality, and desired concurrency. The scheduler and profile model exist so the hardware can change without rewriting the agent, memory, or product layers.

## Failure model

| Failure | Runtime response |
|---|---|
| One model endpoint becomes unhealthy | Remove that instance from admission; keep unrelated capacity available. |
| Checkout capacity is exhausted | Queue or return a structured capacity error; do not bypass the pool. |
| Caller dies while holding a slot | Reclaim the checkout through process monitoring. |
| Tool or generated process fails | Contain the failure in the run/sandbox and preserve diagnostic state. |
| Long task is interrupted | Resume from durable checkpoint and context, not an imaginary live process. |
| Research conflicts with interactive service | Apply priority/claim policy and preserve the research trial for later continuation. |
| Database or scheduler state is unavailable | Refuse or delay effects rather than inventing resource ownership. |

## Operating core and active frontier

The operating platform includes OTP supervision, the Phoenix/PostgreSQL application, local endpoint pooling and health, priority/affinity/profile-aware routing, GPU claims and scheduling, sandboxed execution, the unified execution engine, context/memory, checkpoints, telemetry, and maintenance.

Current expansion is focused on stronger heterogeneous model-placement automation, modality scheduling, cross-run evaluation records, and clearer operational controls. Program discovery, autonomous research strategies, CUDA search, generated tools, and advanced evaluator combinations remain explicitly experimental consumers of the core.

## Limitations

- Self-hosting replaces per-query billing with power, cooling, hardware failure, and operator responsibility.
- V100-class hardware constrains datatype and kernel support relative to newer accelerators; the separate A100 is not automatically interchangeable with the linked V100 pool.
- A sandbox reduces blast radius but does not make arbitrary generated code safe by definition.
- Memory and compaction preserve continuity imperfectly and can retain or omit the wrong information.
- Priority scheduling protects interactive work only when workload profiles and claims are accurate.
- Research rewards can be gamed unless verification remains deterministic and task-specific.

## Relationship to the other systems

- **Flip** can use a HomeCloud provider-compatible endpoint. Flip retains product identity, permissions, tools, provenance, and persistence.
- **Baton** can use HomeCloud-hosted models or sandboxes for worker capacity. Baton retains fleet membership, routing, communication, steering, and harvest.
- **Project Manager** can record experiments and findings produced on HomeCloud. It retains project belief and decision state; HomeCloud retains runtime authority.

## Source

The implementation and host-oriented engineering record are available in the public [HomeCloud repository](https://github.com/wahargis/home-cloud). This portfolio page presents the stable runtime architecture rather than reproducing every tool, provider, or host-specific operational detail.