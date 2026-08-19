# HomeCloud

**A self-hosted AI runtime that turns local accelerators into supervised inference capacity, recoverable agent execution, and a controlled environment for research workloads.**

Owning several GPUs does not by itself produce dependable local AI. A model server can remain reachable while becoming unusable, two workloads can contend for the same memory, a long-running agent can lose its context or workspace, and an experiment can displace the interactive service that justified operating the machine in the first place.

HomeCloud is the operating layer around that hardware. It represents inference as schedulable capacity, routes work through live model slots, gives agent runs owned execution environments and checkpoints, keeps durable context outside any one model window, and places research or media workloads under explicit resource and failure controls.

The system is not defined by one model, one GPU topology, or one autonomous-agent demo. Its purpose is to make local AI services usable as infrastructure when endpoints, accelerators, tasks, and processes have independent lifecycles.

![HomeCloud architecture diagram](assets/diagrams/architecture.svg)

## At a glance

| Concern | HomeCloud’s answer |
|---|---|
| **Who it is for** | An operator or developer running local models and long-lived AI workloads on finite, heterogeneous hardware. |
| **What the user gets** | Healthy endpoint selection, priority-aware request routing, queued workload admission, isolated agent execution, progress and failure visibility, and recovery after interruption. |
| **What the operator controls** | Model-service lifecycle, slot capacity, workload priority, GPU claims, draining and swapping, benchmark isolation, sandbox ownership, checkpoints, and maintenance. |
| **Primary implementation shape** | An Elixir/OTP and Phoenix application with supervised runtime services, PostgreSQL-backed state, local model pools, routing and scheduling components, agent execution, telemetry, and web/MCP-facing control surfaces. |
| **Hardware model** | Capacity is described by model profiles, endpoints, slots, placement, and claims rather than assuming that each physical GPU is one interchangeable worker. |
| **Source boundary** | The implementation and host operations remain private. This case study describes the stable runtime architecture without linking to the private repository or exposing host-specific secrets. |

## The operating problem

A self-hosted system must answer several questions continuously:

- Which model endpoints are healthy, loaded, and capable of accepting another request?
- Which request should receive scarce accelerator capacity now?
- Can a follow-up turn return to the same instance for prompt-cache locality?
- Should an interactive request preempt or delay a background experiment?
- What happens when one endpoint or GPU becomes unhealthy while unrelated capacity remains usable?
- Who owns the workspace, process, container, and cleanup for generated code?
- How does a long task resume after restart, compaction, cancellation, or process failure?
- How can research, image, video, OCR, and maintenance workloads use idle capacity without silently becoming production authority?

HomeCloud treats those as runtime contracts. They are represented in code and state rather than left as shell commands and operator memory.

## A representative day on the system

Consider a local cluster running a user-facing text model while a research program and a media task are also active.

1. A user submits an interactive request. The execution engine asks the inference router for a user-priority slot matching the required model profile.
2. The router consults live pool state rather than a hard-coded GPU count. It can prefer a previously used instance for cache affinity and pins the selected endpoint for the duration of the request.
3. The agent receives a project-derived sandbox, a persistent tool executor where available, bounded setup, and typed progress events. Its commands do not implicitly run in the Phoenix host or another task’s workspace.
4. A research trial requests accelerator time. Its durable state is preserved, but its lower priority and GPU claims prevent it from displacing protected interactive capacity merely because it started first.
5. An image or video request enters a modality-aware queue. The runtime can start the required service, account for conflicting model placement, and expose pending or failed state rather than treating a provider process as instantly available.
6. One local model endpoint becomes unhealthy. The pool removes that instance from admission and can restart or replace it without globally blocking healthy endpoints on other devices.
7. The user-facing task checkpoints its history and engine state. If the process is interrupted, a later run resumes from the freshest durable checkpoint and reconstructs an execution environment; it does not pretend that the original process survived.
8. When work completes or fails, slots, sandboxes, checkouts, and claims return through explicit cleanup paths. Selected facts, evaluation results, or reusable skills may be promoted separately from the transient execution state.

This is the runtime’s value: inference, tools, media, and research can share hardware without sharing authority or failure implicitly.

## Architecture: four operating layers

### 1. Supervised application and service lifecycle

HomeCloud is built as an OTP application. The supervision tree starts the database, PubSub, HTTP clients, model and modality registries, task supervision, inference services, queues, schedulers, execution infrastructure, telemetry, and maintenance processes under explicit restart behavior.

That matters because the runtime is composed of long-lived services with different failure domains. A media queue should not disappear because a browser session fails. An execution log buffer should exist before agents begin writing to it. A GPU scheduler should not infer ownership from an orphaned process. OTP supervision supplies process lifecycle; durable state and domain contracts determine whether work may be resumed or must be failed.

Model service lifecycle can also remain separate from the Phoenix application lifecycle. HomeCloud can adopt externally supervised local endpoints or start managed instances under profile policy. Restarting the application therefore does not have to duplicate GPU-bound servers or confuse endpoint reachability with application process identity.

### 2. Inference capacity and routing

The instance pool manages local model-server instances with checkout/checkin semantics similar to a database connection pool. Each instance records operational properties such as endpoint identity, status, model/profile, context capacity, GPU placement, available slots, health, and draining state.

The implemented pool supports:

- health monitoring and bounded restart behavior;
- multiple concurrent slots per model instance where configured;
- user, research, and background priority classes;
- profile-aware checkout for requests that require a particular model configuration;
- affinity hints so a continuing session can prefer an endpoint with useful prompt-cache state;
- drain and undrain operations for graceful model replacement;
- process monitoring and cancellation cleanup so a caller cannot permanently leak a checkout;
- safe status behavior when the pool itself is unavailable.

The inference router owns the checkout → pin → generate → checkin cycle. Local requests block on the actual pool when all suitable slots are occupied; remote providers use their own concurrency contract. Batch parallelism is derived from live capacity rather than supplied as an arbitrary caller constant.

Failure remains explicit. A checkout error becomes a structured capacity failure. The router does not silently bypass the pool and send a supposedly local request through an unrelated path merely to produce an answer.

### 3. GPU and workload scheduling

Physical accelerators host more than text inference. Research, OCR, translation, image generation, video generation, compilation, profiling, and other CUDA work can compete for memory, processes, and exclusive combinations of libraries or model servers.

HomeCloud represents that competition through several mechanisms:

- durable GPU claims connect workload ownership to scheduler state;
- workload queues admit modality-specific jobs rather than allowing every caller to launch a process immediately;
- priority policy protects user-facing capacity from lower-priority work;
- exclusion and wedge locks prevent combinations known to be unsafe on the installed hardware/software stack;
- idle detection and model-service swapping can reclaim capacity under cooldown rules;
- benchmark locks prevent background scheduling from changing the hardware state during a measured run;
- baseline placement gives displaced hardware an explicit target rather than leaving it in whichever state the last experiment created.

These mechanisms do not make every workload preemptible or every GPU interchangeable. They make the limits visible and place model loading, eviction, waiting, and refusal under policy.

### 4. Agent execution, context, and recovery

The execution engine supports autonomous, interactive, and plan-only modes through one lifecycle. A run can stream typed events for reasoning, tool dispatch, execution, validation repair, completion, and failure, allowing a UI or orchestrator to distinguish what the system is doing rather than showing one undifferentiated loading state.

For source-modifying or tool-using work, the engine owns the execution environment. It can:

- derive a workspace from the registered software project;
- create an isolated agent sandbox and start a persistent executor;
- run bounded setup commands;
- discover or restrict tools by execution profile;
- size parallel task execution from live inference capacity;
- collect tool results and execution telemetry;
- detect repeated actions, repeated output, regression, plateau, or other unproductive loops;
- support cancellation without relying on a growing database log poll every turn;
- checkpoint model history, turn state, loop state, phase, and execution variables;
- resume from the freshest checkpoint across planning or implementation phases;
- perform cleanup from guaranteed terminal paths rather than only after success.

A checkpoint is durable recovery material, not proof that the original process is still alive. On resume, HomeCloud reconstructs the relevant state and environment and informs the model that execution was interrupted.

## Capacity is represented as slots, profiles, and claims

A physical-GPU count is a poor concurrency model.

One text model may span several NVLink-connected V100s. Another may run as independent replicas with several request slots. A media service may occupy a separate accelerator. A quantized model may fit where a full-precision route cannot. A remote provider can expose more HTTP concurrency but different privacy and cost constraints.

HomeCloud therefore reasons about:

```text
model/profile requirement
  + healthy endpoint instances
  + slots and active checkouts
  + placement and GPU claims
  + priority and queue state
  + route policy
  = capacity currently admissible for this request
```

The application using the model does not need to encode the machine’s topology into every request. It asks for a compatible route; the runtime decides whether capacity exists and returns an explicit result.

## Health is local to the failing component

A common operational error is using one global CUDA or service health flag. If one endpoint wedges, such a gate can block otherwise healthy devices and turn partial degradation into total outage.

HomeCloud tracks health at the instance and workload level. The pool can mark one endpoint unavailable, stop assigning new work to it, attempt bounded recovery, and continue admitting requests elsewhere. Pool status calls also return a stable “down” shape when the pool process is unavailable rather than crashing every caller that asks for capacity.

This does not guarantee transparent failover for every model. Context, profile, affinity, and privacy policy may prevent another route from substituting. It does keep the failure boundary honest.

## Interactive service has explicit priority

The runtime differentiates user, research, and background inference. That distinction is not merely a queue label: it controls checkout order and can inform which existing work is eligible to yield or wait.

The GPU workload scheduler similarly treats primary user-facing services as harder to evict than research, OCR, translation, or on-demand media workloads. Swap cooldowns and minimum-uptime rules prevent policy from thrashing models in response to transient queue changes.

An experiment therefore receives capacity because the platform admitted it under current demand—not because it happened to launch a CUDA process first.

## Agent environments are owned resources

Generated code and shell activity need an ownership boundary. HomeCloud associates an execution environment with a task and project, starts tool execution inside that environment, and retains references needed for cleanup and recovery.

The sandbox reduces several classes of failure:

- one agent cannot accidentally use another task’s workspace;
- host configuration is not the default mutation target;
- setup and executor lifecycle become visible runtime state;
- cancellation and failure have a known cleanup owner;
- a resumed run can reconstruct its workspace relationship from durable project/task state.

A sandbox is containment, not a proof that arbitrary generated code is safe. Image provenance, host mounts, credentials, network access, kernel isolation, and admitted tools still require policy.

## Context survives beyond one model window

HomeCloud keeps several forms of context because they serve different retention horizons:

- recent conversation supports the current turn;
- execution history and task state support the current run;
- retrieved documents provide task-specific source material;
- checkpoints preserve recoverable engine state;
- learned facts and reusable skills can survive across selected tasks;
- knowledge relationships and research records retain longer-lived evidence.

The context engine budgets and refreshes that material as work evolves. A large advertised context window does not remove the need to select relevant history, retain provenance, reserve output space, or decide what may be compacted.

Compaction and retrieval can still preserve the wrong information. The architecture makes those operations explicit and inspectable; it does not claim perfect memory.

## Research and autonomous services are consumers of the runtime

The private codebase includes research programs, optimization runners, evaluators, proactive agents, browser tooling, dynamic tools, document processing, OCR, and media services. They are significant consumers of HomeCloud’s scheduling and execution foundation, but they are not all equivalent maturity claims and should not obscure the operating product.

The stable boundary is:

- the **runtime** owns endpoint health, capacity, resource admission, execution environments, context, checkpoints, and failure state;
- a **research workload** proposes candidates, performs trials, compiles or profiles code, and records evaluation results under those controls;
- an **experimental autonomous service** may exercise broader planning or generated-tool behavior without acquiring authority over the scheduler or host merely because it runs inside the same application.

This distinction prevents a long feature inventory from making the platform look less coherent than its implementation actually is.

## Current hardware topology

The current lab configuration combines:

- a four-GPU NVIDIA V100 32 GB NVLink pool with 128 GB aggregate VRAM; and
- a separately attached NVIDIA A100 Drive 32 GB accelerator outside that NVLink fabric.

The runtime does not assume that the five devices form one uniform tensor-parallel pool. Model placement depends on datatype and kernel compatibility, quantization, context memory, interconnect, modality, desired concurrency, and operational stability. Profiles, slots, and claims exist so the hardware can evolve without rewriting the agent and application layers.

Host-specific UUIDs, systemd units, paths, credentials, and service topology remain intentionally outside this public case study.

## Failure model

| Failure | Runtime response |
|---|---|
| One model endpoint becomes unhealthy | Remove that instance from new admission, attempt bounded recovery, and keep unrelated healthy capacity available. |
| Suitable checkout capacity is exhausted | Queue or return a structured capacity error; do not bypass slot ownership silently. |
| Caller exits while holding a slot | Reclaim the checkout through process monitoring and cleanup. |
| A model needs replacement | Drain the instance, allow in-flight work to finish under policy, then switch or restart rather than killing unknown owners. |
| Research conflicts with interactive service | Apply priority, queue, claim, and preemption policy while preserving durable research state. |
| A tool or generated process fails | Contain the failure in the task environment and preserve diagnostic and checkpoint state. |
| A long task is interrupted | Reconstruct from durable checkpoint and project state instead of claiming the old process survived. |
| Scheduler or database state is unavailable | Refuse or delay effects rather than inventing GPU or task ownership. |
| Background swapping would invalidate a benchmark | Hold the declared workloads through a bounded benchmark lock. |
| An asynchronous media service is not ready | Preserve queued/pending/failed lifecycle and start or refuse the service explicitly. |

## Operating core and active frontier

### Operating core

The implementation’s central runtime includes OTP supervision, the Phoenix/PostgreSQL application, local endpoint pooling and health, priority/profile/affinity-aware routing, live slot-derived concurrency, GPU claims and workload scheduling, modality queues, sandboxed execution, execution phases and loop safety, checkpoints and resume, context/memory services, telemetry, and maintenance.

### Active engineering

Current engineering pressure includes stronger heterogeneous placement automation, clearer operator controls, more consistent modality admission, better cross-run evaluation records, and tighter visibility from queue demand through GPU/model state to user-facing progress.

### Experimental consumers

Program search, autonomous research strategies, CUDA optimization, generated tools, advanced evaluators, proactive agents, and computer/browser-oriented workflows remain experimental or workload-specific consumers. Their presence demonstrates pressure on the runtime; it does not convert every research mechanism into a production guarantee.

## Limitations

- Self-hosting replaces per-query billing with power, cooling, hardware failure, dependency management, and operator responsibility.
- V100-class hardware constrains datatype and kernel support relative to newer accelerators; the separate A100 is not automatically interchangeable with the linked V100 pool.
- Slot and priority policy depends on accurate profiles, health, and claims; bad metadata can produce bad scheduling.
- Draining and restart can protect owned requests but cannot make an incompatible model route transparently substitutable.
- A sandbox reduces blast radius but does not make arbitrary generated code safe by definition.
- Checkpoints and compaction preserve continuity imperfectly and can retain or omit the wrong state.
- Research rewards and evaluators can be gamed unless verification remains deterministic and task-specific.
- A broad OTP supervision tree still requires careful dependency ordering and failure-domain discipline; supervision alone is not durable correctness.

## Relationship to the other systems

- **Flip** can consume a HomeCloud provider-compatible endpoint. Flip retains product identity, permissions, admitted tools, provenance, and publication behavior.
- **Baton** can use locally hosted models or sandbox capacity. Baton retains worker harness, route, Run, communication, result, and fleet lifecycle authority.
- **Project Manager** can record experiments and findings produced on HomeCloud. It retains project belief and decision state; HomeCloud retains runtime and hardware authority.

## Source

The HomeCloud implementation and operational repository remain private by design. This case study is grounded in the running code and deployment model but does not link to the private repository or reproduce credentials, production data, host paths, device identifiers, service units, or security-sensitive configuration.