# Project Manager

Project Manager is a system for **keeping long-running research and engineering work intellectually coherent across sessions**.

Most project tools are good at remembering what tasks exist. They are much worse at remembering why a project changed direction, which experiment invalidated an assumption, whether a decision is still supported, or what an incoming agent should trust after weeks of accumulated work. For AI-assisted projects this becomes especially acute: every new session can generate a plausible narrative, but plausibility is not continuity.

Project Manager treats the project itself as a durable evidence structure rather than as a backlog plus chat history.

![Project Manager architecture](assets/architecture.svg)

## The design problem

The system was built around a practical observation: useful project memory has several different kinds of state that should not be collapsed into prose.

A hypothesis is not a finding. A finding is not a decision. A decision is not merely the latest task status. A contradiction should be able to weaken or suspend what follows from it. A dependency should affect what can be done next. A handoff should be derivable from current project state rather than manually rewritten after every session.

Project Manager therefore gives experiments, findings, hypotheses, decisions, constraints, phases, literature, and sessions distinct semantics and links them through typed relationships. The important part is not the number of record types; it is that the project can reason over those distinctions.

## What that changes in practice

Consider a research effort where an early experiment supports an approach, two later findings contradict it, and a design decision was originally justified by the first result. In an ordinary issue tracker, all four objects still exist but their relationship is implicit. A new agent can easily read the old decision and continue executing it.

In Project Manager, the contradiction is part of the project model. The decision retains its rationale and evidence links, downstream work can become questionable or blocked, and the next-session briefing can surface the conflict directly. The goal is not automatic scientific judgment; it is to make changes of belief explicit enough that a human or agent cannot casually paper over them.

This same principle drives phase planning. The system computes actionable work from dependencies and current state rather than assuming the original task order remains correct after the evidence changes.

## Architecture

The implementation is intentionally compact: a Rust domain core over SQLite, exposed through CLI, MCP, and an embedded web interface.

The important architectural choice is that those interfaces are projections of the same model. The MCP server does not get a simplified “agent version” of project state, and the dashboard does not maintain a separate interpretation of progress. The same constraints, graph relationships, and review logic apply regardless of who is interacting with the project.

The graph supports both execution structure and evidentiary structure. Dependency edges answer what can happen next; support, contradiction, derivation, and supersession edges answer why the current project state looks the way it does. Retrieval can then use both text and graph neighborhood instead of returning isolated snippets.

## Session continuity

The strongest use case is agent or human re-entry after context has been lost.

A session can begin by reconstructing the current state of the project: which phase is actually actionable, which hypotheses remain unresolved, which findings materially changed direction, which constraints are active, and which decisions are recent or contested. That orientation comes from the durable graph. A prose handoff can still be generated for convenience, but it is a view over the state rather than the state itself.

This is the central difference between Project Manager and “memory” implemented as transcript retrieval. It stores the project’s evolving epistemic structure, not merely the words previously produced about the project.

## Engineering tradeoffs

SQLite is a deliberate choice. The project is meant to be easy to embed in local agent workflows, inspect directly, migrate deterministically, and use without operating a separate service. The workload is structurally rich but not high-throughput enough to justify distributed storage.

Likewise, the truth-maintenance logic is intentionally bounded. Confidence and contradiction propagation are useful for surfacing when the project’s own evidence no longer agrees with itself; they are not presented as a substitute for human scientific judgment or a universal probabilistic reasoning framework.

## Current maturity

The core model, dependency planning, evidence relationships, contradiction/review logic, retrieval, session continuity, CLI, MCP, and dashboard are implemented. Current work is focused on making causal traversal and newer operations more consistent across interfaces rather than expanding the ontology for its own sake.

The public source is available in the [Project Manager repository](https://github.com/wahargis/project-manager).
