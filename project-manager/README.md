# Project Manager

**A research-control system that preserves what a long-running project believes, why it believes it, and what should happen next.**

Task trackers answer who is doing what. Transcripts answer what was said. Neither is a reliable representation of a project whose direction changes as experiments succeed, evidence conflicts, constraints emerge, and prior decisions become obsolete.

Project Manager (`pm`) gives that evolving reasoning state a durable model. It couples an evidence graph to an execution DAG so human and agent sessions can resume from the project’s current logic rather than reconstructing it from notes.

![Project Manager architecture](assets/architecture.svg)

## The core design decision

Project Manager keeps two related structures separate:

- The **evidence graph** records hypotheses, experiments, findings, decisions, constraints, references, and the causal relationships among them.
- The **execution DAG** records which phases depend on which other work, their current state, and their relative impact.

This distinction matters. A phase can be technically unblocked while its rationale has been contradicted; a finding can remain true while the work it motivated has already been superseded. Treating both as generic tasks or notes loses that meaning.

## A representative project lifecycle

Consider a project testing whether a new retrieval strategy improves long-horizon agent work.

1. The project records a hypothesis and defines an experiment in the phase that can test it.
2. The experiment produces a finding with confidence and provenance.
3. The finding supports the hypothesis strongly enough to justify a decision and unlock a dependent implementation phase.
4. A later experiment finds a counterexample. A contradiction edge lowers confidence and can suspend the affected belief and downstream work.
5. Review surfaces the conflict, the decision remains attached to its original rationale, and the project can branch, supersede the decision, or define a resolving experiment.
6. The next-action engine recomputes which high-impact phase is both dependency-satisfied and epistemically usable.
7. A new human or agent session receives that state as orientation rather than a prose handoff written before the contradiction existed.

The value is not that the system stores more notes. It preserves the causal path by which the project changes its mind.

## Evidence is typed because relationships change behavior

Project Manager uses typed research objects, but the important fact is what the types allow the system to enforce.

An experiment can produce a finding. A finding can support or contradict a hypothesis. A decision must record why it was made and can point to the findings, constraints, or prior decisions that caused it. A later object can supersede an earlier one without deleting the historical rationale. Literature and feedback can enter the same graph without being mistaken for empirical results.

These relationships affect retrieval, confidence, belief state, review, and next-action selection. The graph is therefore part of the control system, not a visualization added after the fact.

## Truth maintenance makes disagreement explicit

Findings, hypotheses, and decisions carry confidence and belief status. Support and contradiction update the target’s evidentiary position; sufficiently strong contradiction can suspend a node and its downstream dependents.

This is deliberately bounded. Confidence arithmetic does not replace scientific or engineering judgment, and the system does not silently rewrite conclusions. It does make incompatible beliefs visible, preserves the evidence on both sides, and prevents an agent from continuing as though the conflict did not exist.

The review layer also looks for structural problems that commonly accumulate over time: important findings with no downstream use, decisions with no causal support, stale constraints, repeated inconclusive work, disconnected literature, and branches that never converge. Repair remains an explicit user or agent action.

## The execution DAG turns current belief into next work

Project phases form a directed acyclic graph with dependencies, impact, and execution state. The scheduler excludes completed or blocked work, orders dependency-satisfied phases, and selects the most consequential next action.

Because this computation reads current project state, the original plan is not treated as sacred. If a supporting hypothesis is suspended or a constraint changes, the next phase can change as well. Repeated failure or lack of progress is surfaced as stagnation rather than allowing the project to cycle silently.

This makes the DAG more than a roadmap. It is the bridge from accumulated evidence to executable work.

## Decisions remain causal records

A decision without its reason is almost useless to a later session. Project Manager requires a non-empty rationale and encourages links to the evidence and constraints that produced the choice.

That record supports questions a normal task tracker cannot answer:

- Was this decision based on measured evidence, a constraint, or a provisional judgment?
- Which later finding invalidated it?
- What downstream work assumed it was still active?
- Was the alternative rejected or merely deferred?

When a decision changes, the system retains the prior reasoning rather than presenting the current state as though it had always been obvious.

## Retrieval returns a project neighborhood, not an isolated note

Text search is useful, but a matching sentence is rarely enough. Project Manager can combine textual relevance with graph relationships, evidence weight, recency, and project scope, then expand the result into nearby findings, experiments, decisions, and constraints.

A session asking about “retrieval recall” should see the experiment that measured it, the finding that resulted, the hypothesis affected, and the decision that followed—not four unrelated search hits.

At session start, the same model supports orientation: active work, high-impact unblocked phases, unresolved contradictions, pending experiments, recent decisions, and expiring constraints. At session end, a durable summary and active-work pointer can be recorded, but the graph remains canonical; the handoff is a projection of current state.

## One domain core, three working surfaces

The Rust core and portable SQLite database serve three interfaces:

| Surface | Why it exists |
|---|---|
| **CLI** | Fast terminal and scripted project work. |
| **MCP stdio** | Direct agent access to the same project, graph, review, and session operations. |
| **Embedded web dashboard** | Human inspection of project state, dependencies, evidence, and structural health. |

All three use the same validation and truth-maintenance rules. MCP is not a weaker agent-only implementation, and the dashboard does not maintain a separate read model that can drift from the project.

SQLite is intentional: the project state remains portable, inspectable, and usable without a network service. That makes Project Manager suitable as a durable companion to local or repository-scoped work while keeping the domain model independent of any one agent harness.

## What is implemented and what remains open

The operating core includes the typed store and migrations, evidence relationships, dependency scheduling, confidence and belief state, contradiction/orphan/staleness review, text and graph-assisted retrieval, session continuity, and the CLI/MCP/web surfaces.

Current work is concentrated on richer causal-backbone traversal, closer parity for newer operations across surfaces, and stronger project-scoped context assembly. More automatic transition and review-gate orchestration remains a deliberate frontier: automation should follow from explicit project state rather than turning the graph into an opaque workflow engine.

## Limitations

- Typed evidence improves continuity but cannot guarantee that a finding is correct or a confidence value is well calibrated.
- Contradiction propagation is a review aid, not a complete formal truth-maintenance system.
- A local SQLite store is portable and simple but not a multi-user collaboration database by itself.
- The usefulness of next-action selection depends on honest dependencies, impact, and current status.
- Agents can still add low-quality or redundant knowledge; graph review and promotion discipline remain necessary.

## Relationship to the other systems

- **Baton** can produce candidate findings, decisions, and run summaries. Project Manager decides what becomes durable project knowledge; it does not supervise Baton's live workers.
- **HomeCloud** can execute experiments and agents. Project Manager records what those experiments mean; it does not schedule GPUs or own sandboxes.
- **Flip** can link product-native sources and community work into a research process. Project Manager does not replace Flip’s social content or authorization model.

## Source

The implementation and deeper reference material are available in the public [Project Manager repository](https://github.com/wahargis/project-manager).