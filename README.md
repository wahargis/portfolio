# William Hargis: Selected Software Systems

This repository contains technical case studies for four implemented software systems. Each case study explains the user-facing system, runtime architecture, important execution and data flows, selected implementation paths, and current limits. The source repositories remain the canonical record for code and detailed project documentation.

## Projects

| Project | System | Main implementation | Case study | Source |
|---|---|---|---|---|
| **Flip** | Hybrid real-time chat and forum platform with AI synthesis, tool use, citations, and generated artifacts | Elixir, Phoenix, PostgreSQL, Oban, React, TypeScript, Electric, Tauri | [Flip case study](flip-technical-overview/README.md) | [wahargis/flip](https://github.com/wahargis/flip) |
| **Project Manager** | Local-first research operations system for phase execution, experiments, evidence, decisions, review, and handoff | Rust, SQLite, MCP, CLI, embedded web UI | [Project Manager case study](project-manager/README.md) | [wahargis/project-manager](https://github.com/wahargis/project-manager) |
| **Baton** | Cross-harness coding-agent orchestration with persistent sessions, multi-member waves, communication, recovery, verification, and integration controls | JavaScript, Node.js, MCP, CLI, Unix-socket resident service, Git worktrees | [Baton case study](baton/README.md) | [Flip-Engineering/baton](https://github.com/Flip-Engineering/baton) |
| **HomeCloud** | Self-hosted AI application platform for inference routing, GPU service lifecycle and claims, agent execution, research optimization, tools, memory, media, and connectors | Elixir, Phoenix, Ash, PostgreSQL, pgvector, local model servers, CUDA | [HomeCloud case study](homecloud/README.md) | `wahargis/home-cloud` (private source repository) |

## Selected work

### Flip

Flip combines synchronous room chat and asynchronous forum discussion in one product. Its synthesis pipeline loads an eligible room-message window, applies quality checks, gives an AI curator bounded access to the source material, and commits validated create-or-append operations to forum threads. The case study also separates command handling, durable client synchronization, transient real-time events, background work, model calls, tools, citations, and artifacts.

[Read the Flip case study.](flip-technical-overview/README.md)

### Project Manager

Project Manager stores execution state and research evidence in one local database. A phase dependency graph determines which work is actionable. A separate typed relation graph connects experiments, findings, hypotheses, decisions, literature, principles, constraints, and feedback. Session context, retrieval, review, audit, and repair operations use those records directly.

[Read the Project Manager case study.](project-manager/README.md)

### Baton

Baton controls full coding-harness sessions across multiple providers. It supports single runs, parallel waves, and declarative multi-member workflows. The system keeps worker questions, replies, checkpoints, shared context, status, and recovery state available to the orchestrator throughout execution. Worker changes are collected as Git results and verified outside the worker worktree before review, adoption, or integration.

[Read the Baton case study.](baton/README.md)

### HomeCloud

HomeCloud is an Elixir-supervised platform for self-hosted AI applications. It routes inference to local or remote providers, coordinates local model services and GPU claims, runs multi-turn agents with tools and memory, and supports long-running research and optimization workflows. The case study follows both an agent request and a compile-test-profile research loop through shared runtime services.

[Read the HomeCloud case study.](homecloud/README.md)

## Repository scope

This is a curated portfolio, not a copy of the source repositories. It includes selected architecture and execution details that can be reviewed without reproducing every command, module, issue, or roadmap item.

HomeCloud source is private. Flip, Project Manager, and Baton link to their public canonical repositories. Licensing and disclosure limits are described in [NOTICE.md](NOTICE.md).
