# HomeCloud

**A local-first GPU inference architecture and custom agent harness for autonomous research and technical work on finite self-hosted hardware.**

HomeCloud combines local model serving, GPU capacity control, supervised agent execution, autoresearch, repository-bound coding work, durable workspaces, documents, connectors, browser and search tools, and media services in one application. The user-facing system is an environment for directing, inspecting, interrupting, resuming, and reusing long-running AI work; the inference and operations layer exists to make that work viable on owned hardware rather than becoming a separate infrastructure console.

The largest body of functionality is autoresearch. Guided research, coding research, and managed project runs can plan work, run experiments, call tools in isolated workspaces, verify outputs, persist findings and artifacts, checkpoint semantic state, and recover after interruption. HomeCloud can operate with Project Manager: Project Manager preserves questions, hypotheses, evidence, decisions, phase dependencies, and handoffs, while HomeCloud supplies model execution, tools, sandboxes, measurement, and GPU control.

The platform is implemented as a supervised Elixir application. Interactive inference, research programs, document workflows, browser automation, OCR, media generation, and external connectors use the same model-routing, resource, execution, persistence, and telemetry substrate. Model-instance pools, quantization-aware GPU claims, request priority, scheduling and eviction, process lifecycle, and checkpoint recovery allow those workloads to share finite hardware without starting unrelated model processes or maintaining incompatible execution state.

## Logical platform architecture

<img src="assets/diagrams/architecture.svg" alt="HomeCloud supervised local-first AI operations architecture" width="100%" />

The logical architecture is independent of one host or accelerator generation. It defines how application requests enter supervised runtime services, how inference and GPU capacity are admitted, how agents receive context and isolated tools, and how results and checkpoints become durable state.

## Reference deployment — August 2026

<img src="assets/diagrams/reference-deployment.svg" alt="HomeCloud four-V100 and A100 Drive reference deployment, August 2026" width="100%" />

The active private deployment provides a concrete implementation of that architecture:

- **four NVIDIA V100 32 GB accelerators** form the principal local inference and research pool;
- a **separate NVIDIA A100 Drive 32 GB accelerator** provides additional heterogeneous capacity;
- model services are health-monitored and exposed through HomeCloud's routing and instance-pool contracts;
- GPU claims and workload scheduling coordinate interactive inference, research, OCR, image, video, and other accelerator-bound work;
- containerized agent workspaces execute tools against durable project state;
- PostgreSQL and durable file storage hold application objects, checkpoints, documents, and artifacts;
- hosted model, search, and media providers remain available as explicit capability routes;
- **Flip is hosted as an application workload and can consume HomeCloud-served local inference while retaining its own community authorization, context, tools, and content authority.**

The diagram does not assert a physical interconnect topology that is not part of the public evidence. It records the deployed accelerator inventory and system roles without turning one hardware arrangement into the definition of HomeCloud.

## Autoresearch and managed project execution

HomeCloud's research layer supports three related forms of work. Guided research develops a question through source gathering, analysis, experiments, and synthesis. Coding research works directly against a repository with file, shell, Git, test, profiling, and benchmark tools. Managed project execution binds those activities to explicit phases, dependencies, success criteria, evidence, and handoffs so the work can continue across sessions and agent processes.

All three use the same execution engine, context system, model routes, tool authority, isolated workspaces, checkpoint format, and lifecycle events. A long-running program can preserve plan and DAG state, trial history, tracked files, context, loop-detection state, score changes, phase markers, and produced artifacts rather than reducing the work to a transcript.

Evaluation can be tied to the object under study. Software and kernel work can use compilation, tests, numerical comparison, profiler traces, and hardware measurements as deterministic evidence. Document and research work can preserve sources, citations, intermediate findings, and review state. The model proposes and interprets work; the application owns execution, measurement, persistence, and recovery.

Project Manager provides an optional durable reasoning layer for these programs. It records the questions, hypotheses, experiments, findings, decisions, constraints, dependency graph, and session handoffs. HomeCloud executes the corresponding research and coding work, returns measured results and artifacts, and supplies the local inference and GPU capacity required by the program.

## Volta/Llama — V100 inference engineering

Volta/Llama is a concrete historical use of HomeCloud's autoresearch layer in conjunction with Project Manager. The project addresses the widening kernel-support gap for NVIDIA V100 and the SM70 architecture in current `llama.cpp`-family runtimes. Its private implementation, `volta_llama`, is an active fork of `ik_llama.cpp` that retargets modern LLM inference work to Volta rather than treating V100 as a compatibility-only backend.

The project includes several related kernel and runtime families:

| Work area | Implemented work |
|---|---|
| **VSIE-F primitives** | PTX-native V100 operations, per-lane emulation of newer load behavior, HMMA `m8n8k4` atom wrappers, online softmax primitives, SM70 shared-memory swizzles, and quantized-value unpacking. |
| **WS-FA FP16-ACC** | Warp-specialized Flash Attention with single-CTA and Split-K variants, bank-conflict-aware shared-memory layouts, packed softmax writeback, and head-dimension support for 128, 256, and 512. |
| **Marlin W4A16 SM70** | A Volta-adapted Marlin development path for quantized matrix multiplication, including correctness-tested bridges and later kernel stages. |
| **PMK-R1** | A skinny Q4_K MMVQ pre-dispatch path for feed-forward down projections and attention output projections. |
| **TurboQuant** | Hadamard-rotation quantization with a GPU inverse-WHT dequantization path. |
| **Multi-GPU topology** | Validation and dispatch work across a single GPU, an NVLink pair, two NVLink pairs separated by PCIe, and a four-GPU 2×2 topology. |

Same-build `llama-bench` measurements on one V100 with Gemma4 26B-A4B Q4_K_M show the default-on WS-FA plus Split-K path improving over the default CUDA Flash Attention path:

| Token-generation benchmark | WS-FA + Split-K | Default CUDA FA | Improvement |
|---:|---:|---:|---:|
| `tg128` | 103.37 | 100.07 | **+3.3%** |
| `tg512` | 93.52 | 83.11 | **+12.5%** |
| `tg2048` | 84.83 | 61.19 | **+38.6%** |
| `tg4096` | 81.51 | 57.16 | **+42.6%** |

The research protocol is as important as the accepted kernels. A claimed improvement must first be shown to reach the intended dispatch path, then pass same-build end-to-end A/B measurement and numerical or output-quality checks. `nvprof` is used to rule out dead dispatch paths; Kullback-Leibler divergence is preferred to tokenizer-sensitive perplexity for output-quality validation.

Rejected approaches remain part of the project record. One Marlin Stage 8 bridge passed numerical correctness checks but was retained as a research artifact after end-to-end diagnostics showed that the approach was dequantization-bound and slower than the existing MMQ dp4a path on the target V100 shape. Project Manager preserves that experiment, finding, and resulting decision; HomeCloud supplies the coding agents, isolated workspaces, local inference, GPU ownership, profiler access, benchmarks, checkpoints, and durable artifacts used to produce it.

## Request lifecycle

A request moves through the system as an operational transaction.

### 1. Classify and admit the workload

The originating surface identifies the workload type, priority, requested capability, model profile, and execution mode. Interactive user work, research programs, and background maintenance do not enter the same undifferentiated queue.

Admission uses current runtime state: configured backends, healthy instances, available slots, GPU claims, workload locks, and service readiness. Callers do not declare arbitrary parallelism that the infrastructure cannot support.

### 2. Resolve the inference route

The inference layer selects a local or remote path based on the required capability and current operating policy. Local model profiles retain their model, quantization, context, topology, and service requirements. Remote providers remain valid routes where their latency, context, tool, or availability characteristics are appropriate.

Higher-level agent and product code uses a stable inference contract. Backend-specific request shapes, health checks, slot semantics, and process behavior remain inside adapters and routing services.

### 3. Check out capacity

The instance pool exposes model servers through checkout and check-in semantics. It tracks healthy instances, available slots, request priority, model affinity, checkout ownership, restart state, draining state, and stable behavior when the pool itself is unavailable.

Batch execution derives concurrency from the number of usable slots. With one healthy slot, work is sequential; additional healthy capacity permits controlled parallelism without every caller implementing its own queue or retry policy.

### 4. Acquire accelerator authority

Model and media services share finite GPUs. HomeCloud represents ownership through claims, queues, locks, demand, and scheduler state. A model is not considered safely replaceable merely because its log is quiet.

The workload scheduler evaluates pending work, active services, idle counters, cooldowns, benchmark locks, spare slot capacity, and configured baseline services before spooling or stopping a heavyweight service. Interactive baselines can remain protected while research, OCR, translation, image, or video workloads use genuinely available capacity.

### 5. Prepare an execution environment

Agent work receives a durable workspace and a containerized sandbox with the required toolchains. The execution engine owns creation, startup, cleanup, and failure reporting. File operations are validated against the mounted workspace; generated workspaces can use quota-controlled durable storage; CPU and memory reservations yield under pressure without preventing use of otherwise idle host capacity.

### 6. Run the agent loop

The execution engine supports autonomous, interactive, and plan-only modes. It assembles context, selects a profile, discovers permitted tools, routes inference, parses and dispatches tool calls, streams phase changes, and records task lifecycle.

Loop controls track repeated actions, repeated outputs, score regression, and stalled improvement. Context and execution complexity can change how much work is allocated without allowing a model to control infrastructure capacity directly.

### 7. Checkpoint semantic state

Long-running work persists more than a log. Phase-aware checkpoints can include:

- message history;
- plan and DAG state;
- context and file-tracking state;
- loop-detection history;
- engine counters and phase marker;
- execution events required for replay or analysis.

A restarted task can resume from the freshest valid phase instead of starting at turn zero or asking a new model session to reconstruct state from printed output.

### 8. Persist the result and release resources

Results return to typed application state. They can become research records, documents, notifications, connector deliveries, media artifacts, or visible product output. Instance checkouts, GPU claims, containers, worktrees, and model-service state are reconciled so subsequent work sees accurate capacity.

## Inference and resource control

### Model profiles and routing

HomeCloud separates the requested capability from the concrete backend serving it. A profile can describe a local text model, an alternate local runtime, a hosted provider, an OCR model, or a media workload. The router evaluates suitability and availability before assigning the request.

This supports local-first operation without assuming local-only execution. Privacy-sensitive or high-volume work can remain on owned hardware; exceptional context, capability, or availability requirements can use a remote route through the same application contract.

### Instance pool

`HomeCloud.Intelligence.InstancePool` manages local inference processes as capacity-bearing resources. It provides:

- priority-aware checkout and cancellation;
- affinity to preserve prompt-cache locality;
- per-instance and multi-slot capacity;
- health checks and restart accounting;
- stable pool-down responses rather than propagating process exits;
- draining and reactive unhealthy reporting;
- status projections for operations and routing.

The pool converts model processes into application capacity that can be inspected, queued, and released.

### GPU workload scheduler

`HomeCloud.Infrastructure.GpuWorkloadScheduler` coordinates service lifecycle against queued work. It can ensure a requested service is running, detect idle lower-priority services, apply cooldowns to prevent thrashing, protect benchmark locks, return displaced hardware to a baseline, and report unavailable services.

This layer prevents application features, research programs, and media tools from independently deciding that the same accelerator may be reconfigured.

## Agent engineering

### Execution engine as the common runtime

The execution engine is shared by autonomous jobs and interactive sessions. Both paths use the same context, routing, tool, loop, sandbox, lifecycle, and telemetry machinery. Product features therefore do not gain a second, weaker agent implementation merely because one request streams to a UI and another runs in the background.

The runtime exposes explicit phases such as reasoning, tool dispatch, tool execution, synthesis, and validation retry. Callbacks can drive the UI while the underlying task retains durable status.

### Tool discovery and execution

Tools are selected by execution profile and can be discovered incrementally rather than sending the entire registry on every turn. The registry covers application operations, files, shell and Git work, testing, web and browser research, memory and graph operations, documents, multimodal generation, and specialized research tools.

The important boundary is not the number of tools. It is that tool schemas, execution, path authority, result normalization, telemetry, and retry behavior are owned by the runtime rather than improvised inside each prompt.

### Context and memory

Context construction combines task state, project state, retrieval, discovered tools, tracked files, token budget, and execution history. Memory and knowledge-graph services can preserve facts, patterns, and relationships across tasks, while compaction prevents long-running sessions from treating the entire transcript as equally important.

### Research and verified evaluation

Research and optimization programs run as managed workloads. They can schedule trials, use lower-priority inference, persist results, and evaluate generated code or kernels through deterministic compilation, tests, numerical correctness checks, profiler evidence, or hardware measurement where appropriate.

The Volta/Llama program demonstrates this path in practice. Project state determines the next experiment; agents modify kernels and runtime code in isolated workspaces; benchmark locks and GPU claims protect measurement; and accepted or rejected results become durable evidence for the next phase.

Research is therefore connected to the same capacity, sandbox, context, telemetry, and checkpoint system as interactive agents. It does not become a separate set of scripts with hidden resource use and incompatible result state.

## Supervised application services

The runtime supports several product domains:

- document storage, indexing, retrieval, and editing;
- browser automation and web research;
- connectors and messaging;
- model installation and service configuration;
- OCR and document extraction;
- image, video, speech, music, and other media workflows;
- research, evaluation, and optimization programs;
- health, infrastructure, and operations views.

These services establish the operational purpose of HomeCloud. The platform exists so multiple applications can reuse one model, resource, agent, and persistence system rather than each embedding a fragile provider-specific stack.

## Reliability and recovery

| Failure mode | HomeCloud behavior |
|---|---|
| **Model process unavailable** | Routing and pool status expose unavailable capacity; callers receive stable errors or alternate routes instead of crashing on a missing GenServer. |
| **Checkout timeout** | Pending requests are cancelled from the priority queue so a slot is not later assigned to a caller that already abandoned the request. |
| **Unhealthy instance** | Health counters, reactive reports, draining state, restart limits, and cooldowns remove the instance from normal availability. |
| **GPU contention** | Claims, locks, demand, idle checks, and the workload scheduler serialize incompatible service ownership. |
| **Agent setup failure** | Lifecycle telemetry is emitted before setup, task status records the failure, and exceptional cleanup handles containers and mounted storage. |
| **Repeated or regressing agent loop** | Action repetition, output repetition, score regression, and plateau detection trigger warnings, reflection, retry, or termination policy. |
| **Process interruption** | Phase-aware checkpoints preserve the execution state required to resume. |
| **Tool path escape** | Sandboxed file operations validate paths against the mounted workspace. |
| **Background starvation of interactive work** | Priority-aware queues and protected baseline services keep user work visible to capacity control. |
| **Deployment variance** | Optional services are configuration-driven and supervised; absent capability is reported rather than assumed. |
| **Invalid optimization result** | Dispatch verification, correctness gates, same-build A/B comparison, and retained negative findings prevent a dead or numerically invalid path from being reported as an improvement. |

## Selected implementation evidence

HomeCloud's implementation repository is private. These paths identify the code responsible for the public architecture.

| Private source path | Implemented responsibility |
|---|---|
| `lib/home_cloud/application.ex` | OTP supervision tree for persistence, model capacity, GPU control, agent execution, research, connectors, media, health, and the Phoenix endpoint. |
| `lib/home_cloud/intelligence/inference_router.ex` and `model_router.ex` | Capability and backend selection across local and remote inference. |
| `lib/home_cloud/intelligence/instance_pool.ex` | Priority checkout, affinity, health, multi-slot capacity, restart, draining, and stable pool-down contracts. |
| `lib/home_cloud/infrastructure/gpu_workload_scheduler.ex` | Queue-aware model-service lifecycle, idle and cooldown policy, benchmark locks, and baseline return. |
| `lib/home_cloud/infrastructure/gpu_claim_registry.ex` | Persistent accelerator claims and ownership coordination. |
| `lib/home_cloud/intelligence/execution_engine.ex` | Autonomous, interactive, and plan-only agent modes; context, tools, model routing, loop safety, sandbox ownership, streaming, and lifecycle. |
| `lib/home_cloud/intelligence/execution_engine/checkpointer.ex` | Phase-aware persistence and restoration of history, plan, context, loop, event, and engine state. |
| `lib/home_cloud/intelligence/agent_sandbox.ex` | Containerized workspaces, durable storage, soft resource controls, tool execution, and path validation. |
| `lib/home_cloud/intelligence/context_engine.ex` | Token budgeting, context construction, file tracking, retrieval, and compaction. |
| `lib/home_cloud/intelligence/tool_registry.ex` | Typed runtime tool registry and profile-aware tool selection. |
| `lib/home_cloud/intelligence/optimization/ttt_discover.ex` | Trial-oriented optimization search, candidate generation, and integration with verified reward signals. |
| `lib/home_cloud/intelligence/optimization/cuda_reward.ex` | CUDA compilation, binary inspection, correctness checks, profiling, measurement validation, and performance scoring. |

The private `volta_llama` repository contains the SM70-specific inference implementation and its benchmark, correctness, dispatch, and topology evidence. The portfolio describes that work without exposing the unavailable repository as a public link.

## Current boundaries

- The public topology documents the four-V100 pool and separate A100 Drive accelerator but omits private hostnames, network layout, credentials, and security-sensitive service configuration.
- HomeCloud is a self-hosted GPU inference, agent, and autoresearch platform, not a general replacement for Kubernetes or a public multi-tenant cloud.
- A private reference deployment demonstrates the system; it does not constrain the logical architecture to one accelerator model or host layout.
- Project Manager integration supplies richer durable project reasoning but is optional; HomeCloud can run research and agent work independently.
- Volta/Llama is a private specialized inference subproject. Public material includes architecture, research method, and selected benchmark results rather than source access.
- Local-first operation does not require every workload to remain local when a hosted route is more appropriate for capability or availability.

[← Back to portfolio](../README.md)
