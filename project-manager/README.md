# Project Manager

Project Manager is a local project-intelligence and execution-control system for research, engineering, and other long-running technical work. It stores project findings, experiments, hypotheses, decisions, literature, constraints, principles, phases, reviews, and handoffs as typed state with explicit relationships.

The system is designed for work that continues across many human and agent sessions. A later session should be able to determine what is currently believed, which evidence supports it, what contradicts or supersedes it, how a result was produced, which branch of work it belongs to, and which execution phase can proceed next.

## System scope

| Area | Current system |
|---|---|
| **Project state** | Portfolios and projects, subprojects, typed nodes and edges, causal and temporal relationships, branches and merges, artifacts, phase state, and session records. |
| **Research and engineering record** | Experiments, findings, hypotheses, decisions, literature, constraints, principles, feedback, reviews, handoffs, and links to files or external material. |
| **Review and truth maintenance** | Explicit contradiction and supersession, confidence analysis for appropriate findings, heuristic contradiction candidates, stale constraints, orphaned state, and project-health views. |
| **Execution control** | Dependency graph for project phases, cycle detection, dependency satisfaction, ready-phase selection, impact ordering, and stagnation checks. |
| **Retrieval and orientation** | Lexical and graph-aware retrieval, typed filters, direct and multi-hop traversal, phase context, branch context, active experiments, and session start and end summaries. |
| **Interfaces** | Shared Rust application and SQLite persistence exposed through CLI, MCP, and web interfaces. |

## Architecture

<img src="assets/architecture.svg" alt="Project Manager research state, review, retrieval, and execution architecture" width="1100" />

The evidence and project graph explains the state of the work. The execution DAG determines which persisted phase dependencies are satisfied. Review, retrieval, session orientation, and agent interfaces operate over both without collapsing them into one structure.

## Representative research and engineering cycle

A project cycle can proceed as follows:

1. **Start a session from current state.** The user or agent loads the active project, current branch, open experiment, pending review items, phase status, and the most relevant connected records.
2. **Record the work as typed objects.** A benchmark run becomes an experiment. Its measurements become findings. A proposed explanation becomes a hypothesis. A constraint or principle is stored separately from a decision that uses it.
3. **Connect the causal and evidentiary relationships.** The experiment produced the finding; the finding supports or contradicts the hypothesis; the decision was informed by selected evidence; an implementation phase depends on the decision; a later review may supersede an older conclusion.
4. **Retrieve the relevant project neighborhood.** Queries can include object type, direct and multi-hop edges, branch, active phase, recency, and lexical content. The result includes why an item is relevant, not only matching text.
5. **Review changes in project belief.** Explicit contradiction, supersession, confidence analysis, stale-state checks, and advisory contradiction candidates identify records that require a decision or correction.
6. **Update execution state.** Completed phases satisfy dependencies. The scheduler identifies dependency-ready phases and orders them using persisted impact data. Review signals can inform a phase change, but they do not silently rewrite scheduler state.
7. **End with a durable handoff.** The session records what changed, what remains open, which evidence was added, which decisions or phases were affected, and what the next session should load.

This cycle supports human work and agent work through the same project record. An agent does not need to infer the entire project from a chat transcript or a directory of notes at the start of every session.

## Typed project model

The store uses a closed vocabulary of project objects and relationships. The exact vocabulary can evolve, but the important distinction is between content and the role that content plays in the project.

Examples of node types include:

- experiment, finding, hypothesis, decision, literature, constraint, principle, feedback, review, handoff, phase, artifact, and research item;
- project and portfolio records that establish scope and hierarchy;
- session and temporal records that preserve when project state changed and which work was active.

Examples of edge semantics include:

- `produced_by`, `supports`, `contradicts`, `supersedes`, `derived_from`, and `informed_by`;
- `depends_on`, `blocks`, `validates`, and links between phases, tests, artifacts, and decisions;
- causal, branch, merge, parent, and provenance relationships used to trace how one line of work affected another.

Typed relationships allow direct questions such as:

- Which experiments produced the findings used by this decision?
- Which current conclusions depend on a finding that was later contradicted?
- What changed on this branch before it was merged?
- Which constraint was active when this design choice was made?
- Which phase is blocked by an unresolved decision or incomplete prerequisite?
- Which records should be loaded for the next session on this subproject?

The graph is persisted project state rather than a visualization generated from nearby text.

## Causal, branch, and temporal history

Technical work frequently has parallel lines of inquiry. Project Manager retains branch and merge relationships so an abandoned or competing path does not disappear when a different direction is selected. Findings and decisions remain associated with the line of work that produced them, and merge records can identify which results were carried forward.

Temporal changes are also retained. Superseding a conclusion does not require deleting the earlier state. A session can identify the current record while still tracing the evidence and decisions that led to it. This supports post hoc analysis, handoff, and correction without treating the latest text as the only project history.

Causal edges are used where the relationship is known. The system does not infer causality merely because two records are close in time or text. Model-assisted suggestions can identify candidate relationships, but durable edges are validated project updates.

## Review and truth maintenance

Project Manager separates explicit project facts from advisory analysis.

An explicit contradiction edge records a known conflict between project objects and can affect belief and confidence views. A supersession edge states that a newer record replaces an older one for current use while preserving the older record in history. Confidence analysis can evaluate suitable numeric findings without assigning false precision to qualitative material.

Contradiction discovery can use structural signals, text markers, numeric divergence, and a second-stage natural-language review to identify candidate pairs. These candidates require confirmation before they become durable project relationships. This preserves high recall without allowing a heuristic or model judgment to rewrite project belief automatically.

Project-health operations also identify state that may require maintenance, including:

- findings or decisions with weak or missing evidence relationships;
- stale constraints that still affect active work;
- orphaned records with no useful project connection;
- incomplete handoffs or sessions without recorded outcomes;
- phases whose state conflicts with their dependencies;
- active work that has not changed for an expected period.

## Retrieval and session continuity

Retrieval operates over project objects and relationships. Lexical matching is useful, but a project query often needs more structure than semantic similarity alone.

A useful result can include:

- the matched record and its type;
- the project, subproject, branch, and session in which it was created;
- supporting, contradicting, superseding, causal, dependency, and test relationships;
- current phase and active experiment context;
- whether the item is current, historical, unresolved, or advisory;
- downstream decisions or phases that may be affected.

Session start and end operations use that state to provide continuity. A new agent session can load current objectives, recent deltas, active experiments, unresolved review items, and ready work. At the end of the session, updates and handoff state are written back to the project rather than left only in the model transcript.

The system therefore supports memory as maintained project state. Vector or lexical search can help find records, but retrieval is not treated as a substitute for typed relationships, lifecycle, and current-state selection.

## Execution DAG

Project phases and their prerequisites form a separate directed acyclic graph. The DAG supports:

- topological ordering and cycle detection;
- persisted phase status;
- dependency satisfaction based on completed prerequisites;
- ready-phase selection;
- impact-based ordering among currently ready phases;
- stagnation and invalid-state checks.

The scheduler is deterministic. Given the same phase graph and status, it returns the same ready set and ordering. This makes the result inspectable and testable.

Review signals remain adjacent to execution state. A contradiction, confidence change, or stale constraint may justify changing a phase, dependency, or decision, but that change is made through an explicit project update. The current implementation does not claim that model-assisted review automatically blocks or rewrites the DAG.

## Agent and interface integration

CLI, MCP, and web interfaces call the same Rust application and persistence model.

The CLI supports direct project operation and scripting. The MCP server exposes structured project operations to coding and research agents, including typed reads and writes, graph traversal, dashboards, reviews, and phase state. The web interface presents project state for human inspection without establishing a second definition of the data model.

Agent access is therefore based on project operations rather than unrestricted database access or Markdown scraping. The application validates node and edge types, project scope, references, and state transitions before persistence.

## Failure and consistency handling

| Problem | System behavior |
|---|---|
| Lost rationale | Decisions remain linked to evidence, constraints, experiments, and earlier decisions. |
| Stale conclusion | Contradiction and supersession preserve the old record and identify the current replacement or unresolved conflict. |
| Invalid graph update | Unknown types, missing references, malformed relationships, and other invalid state are rejected before persistence. |
| Cyclic phase dependency | DAG validation reports the cycle instead of producing a misleading ready phase. |
| Session interruption | Current project state remains in SQLite; a later session can reload active work, deltas, review state, and phase status. |
| Weak retrieval result | The interface can combine lexical results with typed filters and graph expansion instead of treating one ranked text list as complete context. |
| Heuristic contradiction false positive | Candidate analysis remains advisory until an explicit relationship or project update confirms it. |
| Interface drift | CLI, MCP, and web paths use the shared application and storage model rather than independent project semantics. |

## Public implementation

The implementation is available in the **[Project Manager repository](https://github.com/wahargis/project-manager)**.

| Source area | Engineering content |
|---|---|
| [`src/store/mod.rs`](https://github.com/wahargis/project-manager/blob/main/src/store/mod.rs) | SQLite persistence, typed project objects and relationships, lifecycle and provenance state, and store-level validation. |
| [`src/kg/mod.rs`](https://github.com/wahargis/project-manager/blob/main/src/kg/mod.rs) | Direct and multi-hop graph traversal, filters, project neighborhoods, and graph operations used by retrieval and review. |
| [`src/dag/mod.rs`](https://github.com/wahargis/project-manager/blob/main/src/dag/mod.rs) | Topological ordering, cycle detection, dependency satisfaction, ready-phase selection, impact ordering, and stagnation checks. |
| [`src/analysis/contradictions.rs`](https://github.com/wahargis/project-manager/blob/main/src/analysis/contradictions.rs) | High-recall contradiction candidate generation and preparation for deeper language review. |
| [`src/analysis/confidence.rs`](https://github.com/wahargis/project-manager/blob/main/src/analysis/confidence.rs) | Confidence analysis for appropriate numeric finding sets. |
| [`src/cli_runner.rs`](https://github.com/wahargis/project-manager/blob/main/src/cli_runner.rs) | Application-level project operations, session behavior, dashboards, and current phase recommendation logic. |
| [`src/mcp/`](https://github.com/wahargis/project-manager/tree/main/src/mcp) | Structured agent access to the same project model. |
| [`src/web.rs`](https://github.com/wahargis/project-manager/blob/main/src/web.rs) | Human-facing web access over shared project state and operations. |

## Current boundaries

Project Manager is not a general issue tracker and does not attempt to convert every sentence or file into a graph object. It is intended for the project state whose relationships matter across sessions: evidence, experiments, decisions, constraints, phases, review, provenance, and handoff.

Contradiction candidates and confidence results are review inputs, not automatic semantic truth. The deterministic execution DAG does not currently use every review signal as a hard gate. Those boundaries are explicit so later policy work can be evaluated against the current implementation rather than implied as already complete.

[Back to the portfolio](../README.md)
