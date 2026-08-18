# HomeCloud

HomeCloud is a self-hosted AI runtime built to answer a practical question: **what does it take to make local models and autonomous workloads behave like infrastructure instead of experiments?**

The system runs on a four-V100 server, but the hardware is not the point of the project. The point is everything required around the hardware so that local inference can support real applications and long-running agent work: stable process ownership, schedulable capacity, fault isolation, recoverable execution, durable context, and a clear distinction between user-facing service and research workloads.

![HomeCloud architecture diagram](assets/diagrams/architecture.svg)

## Why the project exists

A local model server is easy to start. A local AI platform is much harder.

Once several workloads share the same machine, failures stop being isolated curiosities. A wedged inference process can consume a GPU indefinitely. A long-running agent can leak containers or corrupt its workspace. Research jobs can starve interactive traffic. Restarting the web application can accidentally duplicate or orphan model processes. A large context window can still fail because the wrong information was retained. None of those problems are solved by choosing a better model.

HomeCloud is the systems layer that makes those failure modes explicit and controllable.

## Inference as a schedulable resource

The key abstraction is not “GPU 0” or “model X”; it is a healthy inference slot with an explicit lifecycle.

Requests acquire capacity through a pool, are pinned to a concrete endpoint for the duration of the call, and release that capacity through a guaranteed cleanup path. The scheduler can prefer interactive work, preserve useful model affinity, drain an endpoint before replacement, and remove unhealthy instances from service without treating the entire machine as failed.

This matters because local inference capacity is irregular. One model may span several GPUs; another may run as independent replicas; a media workload may temporarily need exclusive access. HomeCloud represents that topology instead of baking assumptions about it into application code.

The result is that higher-level software can ask for inference without also becoming a GPU process manager.

## Long-running agents as owned executions

The same principle applies to autonomous work.

An agent run owns an execution environment rather than borrowing the host shell. The runtime creates or adopts the relevant sandbox, ties it to persisted task state, records lifecycle events, and cleans it up even when the run fails or is cancelled. Checkpoints and persisted context make recovery possible without pretending that a dead process is still alive.

This is important for source-modifying and research agents because “resume” has to mean more than replaying a transcript. The runtime needs to know which workspace belonged to the task, what state was durable, what external side effects occurred, and what can safely be reconstructed.

HomeCloud’s execution engine therefore treats autonomous, interactive, and plan-only work as different operating modes over one lifecycle model rather than as unrelated code paths.

## Context as managed state

Long-running AI work fails as often from context degradation as from model quality.

HomeCloud does not treat the model’s conversation buffer as the entire memory system. Recent dialogue, task-local working memory, retrieved documents, learned facts, reusable patterns, and graph relationships have different retention semantics. The context engine assembles those sources under a token budget and compacts history deliberately when pressure rises.

The important design judgment is that memory promotion and compaction are controlled transformations of durable state. They are not an excuse to dump every previous token back into the prompt.

## Production service and research on the same machine

HomeCloud also serves as a research environment, particularly for code-generation, search, evaluation, and CUDA optimization work. That work is intentionally subordinate to the platform contract.

Research jobs are useful because they stress the same scheduling, sandboxing, checkpointing, and evaluation machinery under long horizons. They are not used to inflate the platform description with a catalog of experimental algorithms. The durable architectural point is that exploratory workloads can consume spare capacity without becoming indistinguishable from the service that other applications depend on.

This separation also changes how results are trusted. Where a task can be checked deterministically—compilation, tests, correctness comparisons, profiling—the verifier owns the reward signal rather than asking another model to judge whether the first model did well.

## Why Elixir/OTP fits the problem

The platform is built around Elixir/OTP because the hard parts are supervision, isolation, concurrent stateful workers, and failure recovery. Model calls are only one class of process among many.

The application can keep inference endpoints, research runners, schedulers, queues, sandboxes, and maintenance processes under explicit supervision while allowing failures to remain local. PostgreSQL and pgvector provide durable state and retrieval; Phoenix provides the operational and interactive surfaces.

The architecture would survive a change in model family or GPU generation because those are resources managed by the platform, not the platform’s identity.

## Four-V100 deployment

The current machine provides 128 GB of aggregate V100 memory. That constraint is useful because it forces the software to make real decisions about placement, quantization, concurrency, and workload contention rather than assuming effectively unlimited cloud capacity.

Some workloads are distributed across the linked GPUs; others are better served as separate endpoints. HomeCloud’s value is that those choices remain operational policy rather than leaking into every consumer.

## Relationship to the rest of the portfolio

Flip can use HomeCloud as an inference provider without giving HomeCloud authority over users, permissions, or product state. Baton can use it for model capacity or execution environments without giving it source-control authority. That is the intended boundary: HomeCloud owns compute and runtime concerns, not the semantics of the systems that consume them.

## Current maturity

The operating core—supervised local inference, capacity routing, GPU-aware scheduling, sandboxed execution, durable task state, context management, checkpoints, and evaluation infrastructure—is implemented and used as a real local platform. Research modules continue to evolve on top of that base, but they are documented as consumers of the runtime rather than as evidence that the platform needs an ever-longer feature list.
