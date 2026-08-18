# AI systems portfolio

This portfolio is about **how to make AI systems dependable when the model itself is not the authority**.

The four projects here come from different parts of that problem. Flip asks how AI can participate inside a real social product without erasing authorship or bypassing permissions. Project Manager asks how long-running work can retain evidence and decision rationale instead of collapsing into task lists and chat history. Baton asks how capable coding agents can be used as workers without trusting their self-reported state or results. HomeCloud asks how to make local inference and autonomous execution behave like infrastructure rather than a pile of GPU processes.

![Portfolio system architecture](assets/portfolio-system.svg)

## Selected systems

### [Flip](flip-technical-overview/README.md)

A chat-and-forum product built around the idea that live discussion and durable knowledge should be part of one system. The interesting engineering work is not that it can call models; it is the boundary between human authorship, AI authorship, retrieval, product permissions, and durable provenance. Flip is the most product-facing project in the portfolio and the clearest example of treating AI as a governed participant in an existing application rather than as the application itself.

### [Project Manager](project-manager/README.md)

A research and engineering memory system for work that lasts longer than one session. Its purpose is to preserve *why* a project believes something: experiments, findings, contradictions, decisions, dependencies, and changes of mind. The design turns handoff and project orientation into a query over durable evidence rather than another prose summary that becomes stale.

### [Baton](baton/README.md)

A control plane for full coding agents. Baton starts from the premise that a model-driven worker can be extremely capable while still being an unreliable source of truth about its own lifecycle, scope, and correctness. The coordinator therefore owns process state, steering, isolation, recovery, and verification in ordinary code, while the agent is left to do the work that actually benefits from reasoning.

### [HomeCloud](homecloud/README.md)

A self-hosted inference and agent runtime built around a four-V100 server. The point is not the hardware inventory; it is the software layer that makes heterogeneous local AI workloads schedulable, recoverable, observable, and safe to share. HomeCloud is where the portfolio’s model-serving, sandboxing, long-running execution, memory, and research infrastructure are exercised under real resource constraints.

## Common engineering position

Across these projects, I consistently separate **reasoning from authority**. Models are useful where the problem is ambiguous, semantic, or generative. They are not trusted to own permissions, process identity, lifecycle state, irreversible side effects, or the evidence that says their work is correct.

That choice leads to a recurring architecture: durable state instead of transcript memory; explicit ownership instead of convention; independent verification instead of worker self-report; and narrow integration boundaries instead of letting one AI subsystem absorb every concern around it.

The projects are related, but they are deliberately not presented as one grand platform. Each solves a different systems problem and can stand on its own. Their interfaces line up where composition is useful—for example, HomeCloud can provide inference to Flip or execution capacity to Baton—but the responsibility boundaries remain explicit.

For the cross-project architecture and those boundaries, see [Portfolio Architecture](PORTFOLIO_ARCHITECTURE.md).

## How to read this repository

The project pages are written as engineering case studies. They emphasize the problem being solved, the architectural decision that matters, the failure modes that shaped it, and the evidence that the design is real. Implementation details are included when they explain those decisions; exhaustive feature inventories are intentionally avoided.

The [portfolio notice](NOTICE.md) describes the publication boundary for private implementation code and sanitized architecture material.
