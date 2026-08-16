# HomeCloud

**HomeCloud is a self-hosted AI runtime and LLMOps platform that runs local model serving, agent orchestration, GPU scheduling, sandboxed tool use, RAG memory, and checkpointed recovery on four NVIDIA V100 32GB GPUs.**

HomeCloud is a personal, data-center-hardware home lab: one server provides supervised local inference, autonomous research and kernel-engineering workloads, and a production-style environment with no per-query API cost. It is designed for people who want to run AI workloads on their own hardware with the same operational discipline as a small production platform.

![HomeCloud architecture diagram](assets/diagrams/architecture.svg)

## What it does

- **Self-hosted model serving** — local large-language-model inference is supervised as a platform service, independent of the application process lifecycle.
- **GPU scheduling and routing** — inference, research, and background work share the four GPUs with priority-aware scheduling and stable routing.
- **Docker-sandboxed agents** — agent tool execution runs in isolated containers with resource limits, so a bad tool or crashed session cannot disturb the rest of the system.
- **RAG memory** — documents are scanned, chunked, embedded, and searched alongside task memory, learned facts, reusable skill patterns, and a knowledge graph.
- **Context compaction** — long-running sessions stay within token budgets by compacting older context while preserving recent and critical information.
- **Checkpoints and recovery** — sessions and multi-step execution can resume from saved checkpoints, and file-level work can be rolled back scoped to only the steps that failed.
- **Evaluation harnesses** — research work uses task corpora, pass-rate evaluation, metrics collection, and verifier executors.
- **Autonomous research and kernel engineering** — the research lab runs beside production, using lower-priority GPU slots for test-time optimization, CUDA kernel optimization, and long-running research projects.

## How it is organized

The platform has a presentation layer for chat, research, memory, and infrastructure views; an orchestration layer for agent execution and context management; an intelligence layer for model routing, tooling, sandboxing, and memory; and an infrastructure layer for data storage, GPU health, and supervised inference. Each agent session, research trial, and inference request runs in its own process, keeping failures isolated.

## Production and research separation

Production-facing inference is protected from research activity. The primary user-facing model has reserved GPU capacity, while research and on-demand workloads use separate slots and are scheduled below user traffic. The workload scheduler can cycle idle secondary models but never evicts the primary service.

## Reviewer scenario

A good way to evaluate HomeCloud is to think of a long research task: a user submits a question that needs current information and code experiments. The system retrieves relevant documents and prior task memory, runs sandboxed agents that can execute code, keeps the session within a token budget by compacting older context, and checkpoints each turn so the work can resume if interrupted. When the run finishes, the user can inspect the stored results, the memory that was used, and the checkpoints that made the long-running work recoverable — all on local hardware.
