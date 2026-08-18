# Project Manager

**Evidence-aware institutional memory and research control for long-running human and agent projects.**

Project Manager (`pm`) addresses a specific failure mode: a long project cannot be represented faithfully as a list of tasks or a transcript. Over weeks of work, the project accumulates experiments, findings, hypotheses, decisions, constraints, literature, contradictions, and abandoned branches. Without a typed substrate, agents restart without orientation, beliefs drift, contradictory findings coexist, and decisions become detached from the evidence that caused them.

Project Manager stores that work as a small, typed evidence graph backed by SQLite. A single Rust domain core serves a command-line interface, an MCP server, and an embedded web dashboard.

![Project Manager architecture](assets/architecture.svg)

## Architectural thesis

A useful long-horizon memory system must answer four different questions:

1. **What exists?** Typed nodes retain projects, phases, experiments, findings, decisions, hypotheses, principles, constraints, research notes, literature, feedback, and sessions.
2. **Why is it believed?** Typed edges preserve production, support, contradiction, derivation, citation, dependency, and supersession relationships.
3. **What should happen next?** A dependency DAG selects actionable phases by impact and detects stalled execution.
4. **What changed since the last session?** Session and temporal context produce a computed orientation rather than relying on an old prose handoff.

## Knowledge model

### Node types

| Type | Role | State carried |
|---|---|---|
| `Project` | Top-level program or nested effort. | Active, paused, or archived state; aliases and optional parent. |
| `Phase` | Executable unit in the project plan. | Dependency edges, impact, execution status. |
| `Experiment` | A concrete investigation. | Pending/pass/fail/inconclusive status, result, notes, optional hypothesis. |
| `Finding` | Empirical observation. | Confidence and belief status; provenance to upstream work. |
| `Decision` | A selected course of action. | Mandatory rationale plus confidence and belief status. |
| `Hypothesis` | A testable prediction. | Proposed/testing/confirmed/refuted lifecycle and evaluation criteria. |
| `Principle` | Reusable guidance. | Scope and active/superseded/refined state. |
| `Constraint` | A hard operating boundary. | Domain, severity, measured value, and optional expiry. |
| `Research` | Longer-form investigation or reflection. | Work status and project relationship. |
| `LiteratureEntry` | External reference. | URL or identifier, venue/year, findings, reading status. |
| `FeedbackEntry` | Explicit correction or confirmation. | Feedback category and target relationship. |
| `Session` | Cross-session continuity record. | Start/end time, summary, active experiment. |

### Edge types

The graph uses a polymorphic edge table. Any compatible node can connect through relations including:

`ProducedBy`, `Informed`, `Supports`, `Contradicts`, `Supersedes`, `DependsOn`, `RelatedTo`, `CitedIn`, `Contains`, `DerivedFrom`, `TestedBy`, `ViolatedBy`, `BranchesFrom`, and `ConvergesInto`.

The graph is not merely a visualization layer. Edges affect retrieval, confidence, belief status, contradiction analysis, dependency execution, and structural audits.

## Execution and truth-maintenance engines

### Dependency DAG

Phases form a directed acyclic graph. The engine:

- validates and topologically orders phase dependencies;
- excludes blocked or completed work;
- selects the highest-impact actionable phases;
- detects repeated failure and stagnation;
- gives CLI, MCP, and web views the same computed next action.

The project therefore does not depend on an agent remembering the original plan after the evidence has changed.

### Belief revision

Findings, decisions, and hypotheses carry both confidence and belief status. `Supports` and `Contradicts` edges update the target’s evidentiary state. Strong contradiction can suspend the affected node and its downstream dependents instead of leaving incompatible beliefs simultaneously active.

This is a deliberately bounded truth-maintenance mechanism, not a claim that confidence arithmetic replaces scientific judgment. It makes contradiction explicit and reviewable.

### Causal decisions

A decision must include a non-empty `why`. The data model and interfaces encourage links from a decision to the findings, experiments, constraints, or prior decisions that caused it. A decision record is therefore more than a timestamped label.

### Graph health

The analysis layer can surface:

- contradictions and competing branches;
- orphan findings, decisions, hypotheses, and literature;
- stale or expired constraints;
- phases with repeated inconclusive work;
- missing causal links;
- structural graph defects;
- branch and convergence relationships.

Repair operations remain explicit. Detection does not silently rewrite the graph.

## Retrieval and session continuity

Project Manager provides text and graph-assisted retrieval. Search scores can combine textual relevance, evidence weight, graph connectivity, and recency. Context retrieval groups results by type and expands nearby relationships so an agent sees the evidence around a match rather than an isolated sentence.

At session start, the system can assemble:

- active projects and phases;
- highest-impact unblocked work;
- pending experiments;
- untested or contradicted hypotheses;
- recent findings and decisions;
- expiring constraints;
- relevant neighborhood context.

At session end, it can preserve a durable summary and active-work pointer. The graph remains canonical; the handoff is a projection of it.

## One core, three surfaces

| Surface | Role |
|---|---|
| **CLI** | Fast, scriptable project manipulation, review, retrieval, and handoff. |
| **MCP stdio server** | Agent-native access to the same project, evidence, graph, review, repair, and session operations. |
| **Embedded web dashboard** | Human inspection of projects, graph state, execution status, and structural health. |

The surfaces share the Rust domain model and SQLite store. MCP is not a parallel implementation with weaker constraints, and the dashboard does not own separate state.

## Representative workflow

```text
1. Activate project
2. Define phases and dependency edges
3. Record an experiment
4. Add a finding produced by that experiment
5. Link the finding as supporting or contradicting a hypothesis
6. Record a decision and its rationale
7. Run project review for contradictions, orphans, staleness, and stagnation
8. Compute the next actionable phase
9. End the session with a graph-derived handoff
```

A minimal CLI sequence:

```bash
pm project activate atlas
pm phase atlas add "Map retrieval gaps" --impact 40
pm phase atlas add "Test topic briefings" --impact 80
pm next atlas

pm exp atlas add "Evaluate retrieval recall" --phase 1 --status pass
pm finding atlas add "Topic-scoped briefings recovered omitted context" --experiment 1
pm hyp atlas add "Topic briefings improve next-action selection" --phase 2 --finding 1
pm dec atlas add "Adopt topic-scoped briefing" \
  --why "The retrieval experiment improved recall with low operating complexity."

pm review atlas
pm handoff atlas
```

## Implementation structure

| Layer | Responsibility |
|---|---|
| Rust CLI and MCP adapters | Parse commands and protocol calls into domain operations. |
| Typed store and migrations | Enforce schema, lifecycle, uniqueness, and temporal rules. |
| DAG engine | Dependency validation, ordering, next-phase selection, stagnation. |
| Knowledge-graph engine | Traversal, neighborhoods, structural analysis, typed edges. |
| Analysis layer | Confidence, contradiction, orphan, and review logic. |
| SQLite | Portable, durable project state with versioned migrations. |
| Embedded web server | Readable dashboard over the same store. |

## Status

| State | Capability |
|---|---|
| **Shipped** | Typed knowledge store and migrations; project/phase/experiment/evidence objects; DAG execution; confidence and belief state; support/contradiction propagation; contradiction/orphan/staleness review; text and graph-assisted retrieval; CLI; MCP; embedded dashboard. |
| **In progress** | Richer causal-backbone traversal, closer CLI parity for newer operations, and project-scoped retrieval polish. |
| **Planned / experimental** | More automatic transition and review-gate orchestration, richer long-horizon context assembly, and additional migration tooling for older project data. |

## Relationship to the other systems

- Project Manager can retain selected evidence and decisions produced by Baton-controlled work, but it does not supervise workers.
- It can record experiments run on HomeCloud, but it does not schedule GPUs or own sandboxes.
- It can preserve research objects referenced from Flip, but it does not replace Flip’s social content and permission model.

## Source

The implementation is available in the public [Project Manager repository](https://github.com/wahargis/project-manager) under its repository license.
