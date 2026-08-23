# Software and AI Systems Portfolio

These case studies cover four systems I designed and implemented: a community and AI product, a research operations system, a coding-agent fleet manager, and a self-hosted AI runtime. They document user workflows, stored state, background execution, recovery, and implementation references.

## Flip

[![Flip production interface](flip-technical-overview/assets/flip.engineering.png)](flip-technical-overview/README.md)

Flip is a social and creative platform. It includes communities, real-time chat, forums, Personal AI conversations, image and video generation, a searchable media library, an image editor, and self-hosted voice and video rooms.

A message can start a normal discussion, request a Personal AI reply, launch an image or video commission, or become source material for a cited community summary. The platform stores the user-visible request before it starts provider work. Long-running AI and media work continues through durable jobs, updates the original reply or library item, and remains recoverable after a worker restart.

**System:** Elixir, Phoenix, PostgreSQL, Oban, Phoenix Channels, Electric synchronization, React, TypeScript, Tauri, Capacitor, object storage, local and hosted model providers.

[Flip case study](flip-technical-overview/README.md) · [Live product](https://flip.engineering) · [Technical demo](https://flip.tech-demo.dev)

## Project Manager

Project Manager is a local-first system for technical research, experiments, decisions, and project execution. It keeps a project and phase dependency graph beside a typed evidence graph. A benchmark result can be linked to the experiment that produced it, the hypothesis it supports or challenges, the constraint that limits it, and the decision that uses it.

CLI, MCP, and web interfaces call the same typed operations over a SQLite store. Review commands find blocked phases, graph cycles, orphan records, unsupported decisions, contradictions, and expired facts. Repair operations update the same stored model instead of maintaining a separate reporting layer.

**System:** Rust, SQLite, MCP, CLI, web interface, typed graph traversal, validation and repair services.

[Project Manager case study](project-manager/README.md) · [Source repository](https://github.com/wahargis/project-manager)

## Baton

Baton coordinates coding-agent work as runs, workflows, waves, tasks, sessions, worktrees, interaction requests, results, and adoption steps. Operators can start and manage the same run through CLI, MCP, or web interfaces.

The coordinator selects a provider and harness, grants a task-scoped lease, creates an isolated worktree, and records normalized events while the agent works. Tasks can exchange messages and context. Verification and adoption are separate run stages. The coordination store and event history support restart, replay, session recovery, and a complete run timeline.

**System:** Node.js, provider and harness adapters, durable coordination state, workflow interpreter, isolated Git worktrees, CLI, MCP, and web control surfaces.

[Baton case study](baton/README.md) · [Source repository](https://github.com/Flip-Engineering/baton)

## HomeCloud

HomeCloud is a self-hosted AI runtime for a four-GPU NVIDIA V100 node. It routes requests across local inference backends, schedules work by priority and available VRAM, and runs longer agent tasks in isolated workspaces.

The runtime tracks backend health and recent performance before it selects an instance. The GPU scheduler handles memory admission, queue aging, and preemption. Agent tasks use a tool loop with validation, per-turn PostgreSQL checkpoints, a durable sandbox workspace, and resume support. Elixir supervisors restart failed services and keep GPU, model, job, and sandbox state observable.

**System:** Elixir, OTP, Phoenix, PostgreSQL, Docker, NVIDIA V100, vLLM, llama.cpp, SGLang, Ollama, MLX, health-aware routing, GPU scheduling, checkpointed agent execution.

[HomeCloud case study](homecloud/README.md)

## Project boundaries

The four projects are separate systems. Flip is the user-facing product. HomeCloud provides private local inference and agent infrastructure. Project Manager records research and project state. Baton coordinates coding-agent execution. They can be used together, but none of the case studies depends on presenting them as one combined platform.

Project Manager and Baton have public source repositories. Flip and HomeCloud are documented here through product images, system diagrams, and implementation-level case studies without publishing private production data, credentials, messages, or host configuration.
