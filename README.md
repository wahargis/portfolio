# Agent Engineering and AI Systems Portfolio

Four implemented systems for community AI, durable research and coding work, and self-hosted inference. Each combines a user- or operator-facing application with the state, execution, authorization, verification, and recovery systems required for long-lived AI work.

## Projects

### [Flip](flip-technical-overview/)

Flip is a group chat platform and community forum in which an AI assistant participates in chat and curates selected conversation into durable, retrievable forum threads. Direct AI participation and forum curation are separate product workflows: the assistant authors new replies and artifacts in live conversation, while curation organizes human discussion for later reading without erasing who said what or who was allowed to see it.

The assistant is implemented as a first-class product participant rather than an external chatbot attached to the interface. Before a turn, the server resolves the invoking actor, the AI identity, community and room or forum membership, visibility, feature state, and permitted capabilities. The runtime can reply, research internal and external sources, work with documents and data, take product actions, and create image, video, and other artifacts. Typed tool requests pass through the owning domain services, and long-running provider work can continue after the initiating request ends.

Curation retains participant attribution, source-message relationships, source-derived access restrictions, feedback, and recuration history. AI messages, citations, tool results, and generated artifacts also receive durable product identities and lifecycle state.

**[Flip technical case study](flip-technical-overview/)** · **[Conversation-to-forum lifecycle](flip-technical-overview/diagrams/conversation-to-forum.svg)**

### [Project Manager](project-manager/)

Project Manager is a local-first research and technical project system for work that unfolds across many sessions, experiments, and changes of direction. It keeps the project's questions, evidence, hypotheses, decisions, constraints, tasks, artifacts, branches, and handoffs in one durable record, so a person or agent can resume from the current state of the work instead of reconstructing it from chat history, notes, or issue comments.

The system separates the evidence model from the execution plan. Support, contradiction, supersession, derivation, causal, branch, merge, and provenance relationships explain why conclusions changed. A deterministic phase DAG identifies dependency-satisfied work. Session operations assemble the relevant project subgraph at the beginning of work and write a durable handoff at completion. A Rust CLI, MCP server, and embedded web application operate on the same persisted state.

This structure supports research, architecture, implementation, and evaluation work in which a failed experiment, changed hypothesis, or superseded decision remains part of the usable project history rather than disappearing from the current prompt.

**[Project Manager technical case study](project-manager/)** · **[Typed project record](project-manager/assets/project-record.svg)** · **[Public source](https://github.com/wahargis/project-manager)**

### [Baton](baton/)

Baton is a control system for long-running coding work distributed across multiple agent harnesses, workers, Git worktrees, and review stages. It starts from an engineering outcome and turns it into a durable run that can plan work, select compatible harness and model routes, release dependency-ready tasks, preserve shared context, recover detached sessions, collect results, and decide what to review, adopt, and integrate.

Baton does not flatten each coding harness into a stateless completion API. Provider adapters preserve native session identity, interaction, approval, interruption, checkpoint, and recovery behavior. The controller owns run and worker journals, leases, inboxes, questions, worktree assignments, result contracts, reviews, integration decisions, and cleanup. Worker completion and workflow completion remain separate: a run succeeds from required artifacts and verification state, not from a worker's statement that it finished.

The same run model is available through embedded, resident, CLI, and MCP surfaces, allowing a human operator or AI orchestrator to coordinate the same durable state without separate control implementations.

**[Baton technical case study](baton/)** · **[Run lifecycle](baton/assets/run-lifecycle.svg)** · **[Public source](https://github.com/Flip-Engineering/baton)**

### [HomeCloud](homecloud/)

HomeCloud is a local-first GPU inference architecture and custom agent harness for running autonomous research and technical work on finite self-hosted hardware. It combines local model serving, GPU capacity control, supervised agent execution, research and coding workflows, durable workspaces, documents, connectors, browser and search tools, and media services in one application.

The central body of work is its autoresearch system. Guided research, coding research, and repository-bound project runs can move through explicit phases, call tools in isolated workspaces, run verified experiments, persist results, checkpoint semantic state, and recover after interruption. Project-bound programs can use Project Manager as the durable reasoning and planning layer while HomeCloud supplies model execution, tools, measurement, workspaces, and hardware control.

The infrastructure beneath those workflows manages model-instance pools, quantization-aware GPU claims, request priority, scheduling and eviction, local and hosted provider routes, process health, leases, and application telemetry. The reference deployment operates a four-V100 inference and research pool with a separate A100 Drive accelerator, but the architecture is defined by its resource and execution contracts rather than by one host layout.

**[HomeCloud technical case study](homecloud/)**

#### Volta/Llama inference engineering

Volta/Llama addresses a practical inference gap: current `llama.cpp`-family kernels increasingly target newer NVIDIA architectures, leaving V100 systems without equivalent attention, quantized matrix, and multi-GPU execution paths. The private `volta_llama` project is an active Volta-focused fork of `ik_llama.cpp` that retargets modern inference work to SM70. It includes PTX-native VSIE primitives, warp-specialized Flash Attention with FP16 accumulation and split-K execution, a Volta-adapted Marlin W4A16 path, skinny Q4_K MMVQ dispatch, TurboQuant, and topology-aware operation across one GPU, an NVLink pair, PCIe-split pairs, and a four-GPU 2×2 layout.

Same-build `llama-bench` measurements on a single V100 with Gemma4 26B-A4B Q4_K_M show the default-on WS-FA plus Split-K path improving over the default CUDA Flash Attention path by 3.3% at `tg128`, 12.5% at `tg512`, 38.6% at `tg2048`, and 42.6% at `tg4096`. The project verifies that optimized dispatch is active before accepting a result, compares end-to-end performance from the same build, checks numerical correctness, and uses Kullback-Leibler divergence rather than tokenizer-sensitive perplexity for output-quality validation.

Within this portfolio, Volta/Llama is the clearest historical demonstration of HomeCloud autoresearch operating with Project Manager. Project Manager preserves hypotheses, experiments, findings, decisions, dependencies, and session handoffs; HomeCloud supplies agent execution, isolated workspaces, local inference, GPU scheduling, hardware benchmarks, and durable research results.

## Engineering coverage

| Area | Portfolio work |
|---|---|
| **Agent runtime design** | Product-event admission, actor and object scope, context construction, retrieval, model routing, tool loops, effects, terminal validation, continuation, and recovery. |
| **Multi-agent orchestration** | Goal and plan contracts, task topology, waves, workflows, route capability, persistent sessions, questions, approvals, waits, interruption, steering, shared context, harvesting, and run lifecycle. |
| **Durable project reasoning** | Typed evidence, causal and branch history, contradiction and confidence review, provenance, graph-aware retrieval, temporal state, phase dependencies, and session handoff. |
| **AI infrastructure and inference** | Supervision, model processes, capacity pools, priority queues, GPU ownership, workload transitions, local and remote routing, SM70 kernel optimization, multi-GPU topology, and hardware evaluation. |
| **Application and data systems** | Phoenix and OTP applications, Rust services, Node.js control systems, PostgreSQL, SQLite, Oban, realtime channels, synchronization, CLI, MCP, web, desktop, and mobile surfaces. |
| **Reliability and evaluation** | Idempotent effects, explicit lifecycle state, bounded retry, stale-state rejection, restart recovery, artifact verification, deterministic tests, route-specific model evaluation, numerical validation, and operational telemetry. |

## Representative system evidence

- Flip documents a direct AI turn and a separate conversation-to-forum lifecycle with source relationships, operation state, attribution, and recuration.
- Project Manager includes the public `atlas` quickstart and a broader typed project record connecting phases, experiments, findings, hypotheses, decisions, constraints, sessions, and review.
- Baton includes a recorded two-member wave in which both sessions reached `work_completed`, one required result was missing from the harvest, and the workflow correctly ended incomplete; the lifecycle diagram shows how a normal run proceeds through capture, verification, review, and integration.
- HomeCloud documents the four-V100 deployment, separate A100 Drive accelerator, model-instance capacity, GPU scheduling, agent execution, checkpointing, and application-service flow.
- Volta/Llama adds a concrete kernel-engineering record: measured same-build Flash Attention improvements, SM70-specific kernel families, topology-aware validation, correctness gates, and rejected optimization paths retained as research evidence.

## System boundaries

- Flip, Project Manager, Baton, and HomeCloud are independent primary systems. Volta/Llama is a specialized inference-engineering subproject under HomeCloud.
- Flip, HomeCloud, and Volta/Llama are private implementations. Their public technical material is contained in this portfolio and does not link to unavailable source repositories.
- Project Manager and Baton link to their public implementation repositories from their project pages.
- HomeCloud and Project Manager can be used together for persistent autoresearch, but neither system requires the other to operate.
- Public material omits product data, credentials, provider keys, private messages, host-specific secrets, prompt and persona configuration, and administrative authority.

The systems' responsibility boundaries and possible standards-based integration surfaces are summarized in **[System boundaries and interfaces](PORTFOLIO_ARCHITECTURE.md)**.

## Live systems

- [Flip product](https://flip.engineering)
- [Synthetic Flip technical environment](https://flip.tech-demo.dev)
