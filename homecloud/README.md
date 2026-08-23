# HomeCloud

HomeCloud is a supervised local-first AI application runtime. It operates model services, GPU capacity, agent execution, research workflows, documents, connectors, browser and search tools, and multimodal workloads as parts of one Phoenix and OTP application.

The current deployment is built around four NVIDIA V100 32 GB accelerators. The software does not treat that hardware as four independent model processes. It maintains model-instance capacity, workload priority, GPU ownership, service transitions, agent workspaces, checkpoints, health, and application state so several AI capabilities can share the machine without each feature implementing its own process and resource control.

## System scope

| Area | Current system |
|---|---|
| **Supervised application** | OTP supervision for database access, PubSub, HTTP clients, model services, instance pools, GPU monitoring and scheduling, agent processes, research, connectors, browser and search services, model installation, health, notifications, media services, and Phoenix endpoints. |
| **Inference runtime** | Local and remote model adapters, route selection, model-instance checkout and return, slot capacity, affinity, priority queues, health, restart state, throughput and telemetry. |
| **GPU operations** | Hardware monitoring, explicit claims, queue state, workload ownership, idle and cooldown checks, controlled model-service transitions, locks, and return to baseline capacity. |
| **Agent execution** | Autonomous, interactive, and plan-only modes; context assembly; tool discovery by profile; model and tool loop; streaming events; repetition, regression, plateau, and turn limits; lifecycle and result persistence. |
| **Isolation and recovery** | Containerized workspaces, validated file paths, durable workspace storage, soft resource controls, cleanup, phase-aware checkpoints, and resumable messages, plans, context, loop state, and engine state. |
| **Application services** | Research and evaluation, document and file workflows, connectors, browser and search tools, model management, OCR, notifications, image and video services, and other local AI application surfaces. |

## Architecture

<img src="assets/diagrams/architecture.svg" alt="HomeCloud supervision, inference capacity, GPU control, agent execution, tools, checkpoints, and application services" width="1100" />

The diagram separates application requests, resource admission, and agent execution. A model request reaches the execution engine only after route, instance, and GPU state permit it. Agent state and application results are persisted independently of the model-server process.

## Supervised application runtime

HomeCloud runs as an OTP application rather than a collection of shell scripts. The supervision tree starts core database and communication services, model and infrastructure controllers, tool and modality registries, agent and research supervisors, connectors, browser services, media queues, health monitoring, and the Phoenix endpoint according to configuration.

This structure provides explicit process ownership and restart behavior. Optional services can be enabled or disabled without making the rest of the application assume that every model backend, connector, browser, or media provider is present. Process failure becomes visible application state and telemetry instead of an unexplained missing endpoint.

The supervision tree also determines which component owns each long-lived responsibility. Model-server lifecycle belongs to the model and instance control layer. GPU changes belong to the claim and workload scheduler. Agent tasks belong to the agent and execution supervisors. Application features request these services through stable interfaces instead of managing operating-system processes and hardware independently.

## Workload classes and admission

HomeCloud serves different kinds of work:

- interactive requests from a user-facing application;
- long-running autonomous or interactive agent tasks;
- research, benchmark, optimization, and evaluation jobs;
- document, search, connector, browser, and data workflows;
- image, video, OCR, and other multimodal operations;
- background maintenance and model-management work.

These workloads do not have the same latency, duration, model, tool, or resource requirements. Admission considers workload class, priority, selected route, model capability, healthy instance capacity, current queues, and GPU ownership.

User work has explicit priority over research and background work. Lower-priority tasks can use spare capacity without being allowed to make interactive service unpredictable. Batch concurrency is derived from healthy available inference slots rather than a caller-provided number that may exceed the machine's actual capacity.

A request that cannot obtain the required route, instance, or GPU state remains queued or fails with a visible reason. It does not start a competing model process or assume that an accelerator is free because recent utilization is low.

## Model routing and instance pools

Higher-level services use a stable inference interface while adapters handle local and remote backend differences. Route selection can consider modality, context requirements, tool support, privacy, latency, cost, and current health.

Local model servers are managed as instance pools. The pool tracks:

- available and busy slots;
- which model or route a slot serves;
- workload priority and wait queues;
- model or session affinity where reuse is useful;
- health and restart state;
- checkout ownership and return;
- aggregate healthy capacity used to set parallelism.

Checking out an instance creates a temporary ownership relationship between the request and a healthy slot. Returning the instance updates queue and capacity state. If a process becomes unhealthy, the slot is not left available only because its record still exists.

Remote providers can be used through explicit adapters when their capabilities or operating properties are appropriate. They are not presented as identical to local capacity: provider availability, cost, privacy, rate limits, and tool or modality support remain part of routing and telemetry.

## GPU ownership and workload scheduling

GPU control is represented as application state. Monitoring alone is insufficient because low utilization does not prove that a model process or workload has released its claim.

The GPU layer maintains hardware state, claims, workload ownership, queue state, locks, idle thresholds, and cooldown periods. The workload scheduler can preserve a configured interactive baseline, wait for the hardware and request queues to become safely idle, transition a model service for an admitted workload, and return the machine to baseline after work drains.

A service transition is therefore a controlled state change:

1. identify the requested workload and required accelerator set;
2. verify route and model configuration;
3. inspect claims, active requests, queues, utilization, and cooldown state;
4. acquire the required lock and workload ownership;
5. stop or change model services through the designated controller;
6. verify the new service health before admitting work;
7. track completion and release;
8. reconcile the previous or baseline service when capacity is no longer needed.

This prevents independent subsystems from starting incompatible processes on the same accelerators or treating a partially stopped service as free capacity.

## Agent execution engine

The execution engine supports autonomous, interactive, and plan-only tasks. It prepares an execution from persisted application state rather than sending one prompt directly to a model endpoint.

A task can include:

- selected messages and application records;
- a plan and phase-specific state;
- a model route and request limits;
- a tool profile resolved from the current task and policy;
- a containerized workspace and durable file area;
- lifecycle, progress, and result subscribers;
- turn, repetition, plateau, quality, time, and resource limits.

During execution, the engine dispatches model turns, parses tool requests, invokes permitted tools, records results, emits phase and lifecycle events, and evaluates whether the task is progressing. It tracks repeated actions, repeated output, score regression, stalled improvement, and turn limits so an agent loop can stop or surface a warning before consuming unbounded time and compute.

The engine also supports infrastructure-derived batch execution. It uses healthy instance capacity and workload policy to determine how much parallel work can run, rather than assuming that all configured GPUs or model services are available to every task.

## Context, tools, and application state

Context is assembled from the application and task rather than inferred only from a model transcript. It can include stored messages, plans, prior tool results, research records, documents, files, connector data, browser results, model and infrastructure status, and phase-specific checkpoint data.

Tools are discovered through profiles and registries. A task receives the tools appropriate to its purpose and workspace. Browser, search, file, code, document, media, and other services remain application capabilities with their own validation, lifecycle, and result state.

The tool broker and registries allow new application services to reuse the same execution, telemetry, and policy infrastructure. This is the main reason HomeCloud includes product services above raw inference: a document workflow, research job, image operation, or connector task should not ship another independent model process, tool loop, and recovery mechanism.

## Sandboxes and workspace control

Agent tasks use containerized workspaces with selected toolchains. The runtime creates or attaches the workspace, mounts durable task storage, validates file operations against the assigned root, applies soft resource controls, and owns cleanup.

The container boundary reduces accidental host mutation and gives different tasks reproducible tool environments. It does not by itself grant authority to external services or protected application data; those capabilities still require an admitted tool and application-level scope.

Workspace state can outlive one model turn or engine process. This supports multi-phase work, inspection after failure, and resumption without reconstructing every generated file from the conversation history.

## Checkpoint and recovery model

Long-running agent work stores more than logs. Checkpoints can include:

- message and turn history;
- current plan and phase;
- context variables and selected records;
- tool and event state;
- repetition and loop-detection state;
- quality or score history;
- engine mode, limits, and progress;
- references to the durable workspace and result records.

Phase-aware snapshots allow planning, implementation, critique, research, or other stages to retain separate current state. After a process interruption, the runtime can select the freshest valid checkpoint and resume the execution model instead of replaying every turn from the beginning.

OTP supervision restarts failed processes, while persisted task and checkpoint state determines whether the work should resume, fail, or wait for operator action. Model-server restart and agent-task recovery are separate concerns and can proceed independently.

## Research, evaluation, documents, connectors, and media

HomeCloud's research and evaluation services use the same routing, capacity, sandbox, event, and persistence infrastructure as interactive agent work. Lower-priority research can run when capacity permits and stop yielding to user work without requiring a separate batch platform.

Document, connector, browser, search, OCR, image, and video features operate as application workloads over the shared runtime. Their requests and artifacts remain durable product state. A media or document job can continue asynchronously, report progress, fail with diagnostic state, and return a stored result to another application workflow.

This common runtime also supports experimentation with model routes, prompts, tools, optimizers, code and CUDA work, and other local AI research. Experimental workloads remain distinguishable from interactive application services through priority, routing, and lifecycle state.

## Failure and recovery

| Problem | System behavior |
|---|---|
| Model process fails health checks | The instance becomes unavailable, restart state is recorded, and queued work is not assigned until health is restored. |
| GPU ownership is disputed | Claims, locks, active requests, and queue state block a service transition until ownership is resolved. |
| Research competes with interactive work | Priority queues and baseline policy preserve user-facing capacity and defer lower-priority jobs. |
| Agent repeats actions or output | Loop, plateau, regression, and turn controls warn or stop the execution according to policy. |
| Tool or setup fails | Lifecycle and tool results remain stored; cleanup runs on exceptional paths and the task becomes failed, retryable, or resumable. |
| Execution process restarts | Persisted task state and phase-aware checkpoints provide the resume point. |
| Container or workspace is missing | Workspace validation fails before tool execution or resumption; the task is not silently continued in a new arbitrary directory. |
| Optional service is not configured | Capability and health state keep unavailable connectors, models, or media services out of admitted routes and tools. |
| Remote provider fails | Route policy can select a compatible alternative or return a visible failure without changing local GPU ownership. |
| Work completes but capacity is not released normally | Checkout, claim, scheduler, and reconciliation state support cleanup and return to baseline. |

## Current implementation and boundaries

The current implementation includes the OTP application structure, model and instance control, GPU monitoring and workload scheduling, agent execution modes, context and tool infrastructure, containerized workspaces, checkpointing, research and evaluation services, documents, connectors, browser and search tools, and multimodal application services.

HomeCloud is not a public multi-tenant cloud scheduler or a replacement for Kubernetes. It is a self-hosted application and operations platform for owned AI hardware and selected remote services. Hardware- and provider-specific configuration remains part of the private deployment and is not exposed in this portfolio.

The implementation repository is private. This page and its diagram are the public technical description and do not link to private source paths, credentials, host identifiers, model-service names, provider keys, or administrative endpoints.

[Back to the portfolio](../README.md)
