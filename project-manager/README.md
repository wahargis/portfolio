# Project Manager

**A research and engineering control system that preserves what a long-running project currently believes, why it believes it, and what work is justified next.**

Task trackers answer who is doing what. Notes and transcripts answer what somebody wrote or said. Neither is a reliable representation of a project whose direction changes as experiments succeed, measurements disagree, constraints emerge, decisions are superseded, and work passes between human and agent sessions.

Project Manager (`pm`) gives that evolving state a durable model. It records evidence and causal relationships, keeps execution dependencies separate from belief, surfaces contradictions and structural gaps, preserves session continuity, and computes which phase is both unblocked and worth doing next.

The knowledge graph is an important implementation plane, but it is not the whole product. Project Manager is the loop connecting **evidence, belief revision, decisions, execution, review, and handoff**.

![Project Manager architecture](assets/architecture.svg)

## At a glance

| Concern | Project Manager’s answer |
|---|---|
| **Who it is for** | Researchers, engineers, and coding/research agents working on projects that last longer than one session and change direction as evidence accumulates. |
| **What it replaces** | Ad hoc combinations of task lists, transcripts, scratch notes, handoff summaries, and undocumented decisions that cannot explain the project’s current logic. |
| **What it preserves** | Projects, phases, hypotheses, experiments, findings, decisions, principles, constraints, literature, feedback, sessions, confidence/belief state, and typed relationships among them. |
| **What it computes** | Dependency-satisfied next phases, impact ordering, stagnation, stale or disconnected state, graph neighborhoods, and review material for contradictions and unsupported decisions. |
| **Implementation shape** | A Rust domain core with SQLite persistence and shared CLI, MCP stdio, and embedded web access. |
| **Source** | Public in the [Project Manager repository](https://github.com/wahargis/project-manager). |

## The problem it solves

Consider a project evaluating a new retrieval strategy for long-running agents. The original plan says to implement it after a benchmark phase. The benchmark produces an encouraging result, a decision is made, and implementation begins. Two weeks later another experiment shows that the gain disappears at larger context sizes.

A task tracker may still show the implementation phase as active. A transcript may contain both conclusions somewhere. A generated handoff may repeat whichever conclusion happened to be most recent in its context. None of those representations automatically preserve that:

- the first decision was reasonable under the evidence available at the time;
- a later finding conflicts with the assumption that justified it;
- dependent work may now require review rather than blind continuation;
- the project needs a resolving experiment or a revised decision;
- the next useful action may have changed.

Project Manager stores those facts as project state rather than asking every new session to infer them again from prose.

## A working session, end to end

A typical project cycle looks like this:

1. **Orient.** A person or agent opens the project and receives active phases, recent decisions, unresolved contradictions, pending experiments, current constraints, and the highest-impact unblocked work.
2. **State the claim.** The session records a hypothesis, prediction, and criteria rather than burying the idea in a transcript.
3. **Run and record.** An experiment produces findings, measurements, notes, and a terminal status. Findings retain their relationship to the experiment that produced them.
4. **Connect meaning.** Typed edges record whether evidence supports, contradicts, tests, derives from, informs, or supersedes another project object.
5. **Decide.** A decision records what was chosen and why, with links to the evidence and constraints that justified it.
6. **Revise explicitly.** New evidence can create a contradiction or supersession relationship without deleting the earlier rationale or pretending the project always knew the current answer.
7. **Recompute work.** The execution DAG identifies dependency-satisfied phases, excludes completed or deprioritized work, and orders the remainder by impact.
8. **Review and hand off.** Structural review finds stale hypotheses, disconnected findings, unsupported decisions, unresolved branches, or repeated failure. Session state records where work stopped, but the typed project model—not the handoff prose—remains canonical.

The value is not that the system stores more notes. It preserves the causal path by which a project changes its mind and turns that current understanding into executable work.

## Three coordinated models

### 1. Evidence and decision state

The evidence model contains typed objects rather than one generic note table:

- **hypotheses** state claims to test, with predictions, criteria, status, and confidence/belief fields;
- **experiments** represent bounded attempts with status, result, and links to the phase or evidence that motivated them;
- **findings** preserve observations and measurements produced by experiments;
- **decisions** record both the choice and its rationale;
- **constraints** identify hardware, software, or process limits and can retain source, severity, measured value, and expiry;
- **principles, literature, research, and feedback** enter the same project model without being mistaken for empirical findings.

Typed edges carry causal meaning. The implemented relation set includes support, contradiction, supersession, dependency, derivation, testing, violation, citation, containment, branching, and convergence. Those relationships can change retrieval, review, belief state, and downstream interpretation; they are not merely labels for drawing a graph.

### 2. Execution state

Project phases form a separate directed acyclic graph. Each phase carries project identity, dependencies, impact, status, goals, success criteria, and timing. The DAG engine can topologically order work, identify dependency-satisfied phases, exclude completed or deprioritized phases, and rank actionable work by impact.

Keeping execution separate from evidence prevents two common category errors:

- a phase can be technically unblocked while the belief that justified it is under contradiction review;
- a finding can remain historically valid even after the implementation phase it motivated has been superseded or completed.

The execution graph is therefore a bridge from current project understanding to work, not a replacement for the evidence model.

### 3. Temporal continuity and structural review

Sessions, temporal deltas, staleness reports, search results, and velocity measures let a new session ask what changed rather than reread the whole database. Review operations can surface stale hypotheses and experiments, unconnected findings, contradictory records, orphaned knowledge, repeated failed or inconclusive work, and project structures that no longer lead to a decision or next action.

This is what makes Project Manager useful across long time spans. A session summary is a convenience projection. The underlying project objects and relationships remain the durable explanation.

## Evidence is typed because relationships change behavior

An experiment can produce a finding. A finding can support or contradict a hypothesis. A decision can be informed by one or several findings and constrained by available hardware or process. A later decision can supersede an earlier one without erasing why the earlier choice was made. Literature can motivate research while remaining distinguishable from locally observed results.

These distinctions support questions that ordinary task software cannot answer reliably:

- Which measurements justified this decision?
- What later evidence weakened or contradicted it?
- Which active phases depend on assumptions now under review?
- Is this finding unused, or did it actually affect project direction?
- Was an alternative rejected, made infeasible by a constraint, or merely deferred?
- Which experiment would resolve the current disagreement?

The graph is valuable because it preserves those semantics, not because a graph visualization is inherently sophisticated.

## Contradiction and truth maintenance are explicit review mechanisms

Project Manager supports both recorded contradiction edges and active contradiction review. The current candidate detector is designed for high recall: it compares negation, explicit correction markers, domain-tuned antonyms, and materially divergent numbers with overlapping context. It can prepare the resulting candidates for a second semantic classification step rather than pretending a lexical heuristic has established truth.

Confirmed relationships remain typed project state. Support, contradiction, and supersession can update the evidentiary position of affected objects under the store’s truth-maintenance rules, while the original evidence and rationale remain inspectable.

This is deliberately bounded. The system does not claim to solve scientific truth, and it should not silently rewrite conclusions. Its job is to prevent incompatible project beliefs from remaining invisible and to make the required review or resolving work explicit.

## Measurement confidence is a signal, not an oracle

For experiments with repeated numeric findings, the implementation can extract compatible units and compute a Median Absolute Deviation–based consistency signal. This is useful for distinguishing a tight set of measurements from a result dominated by noise and for prompting a rerun when evidence is weak.

The score is not a universal statistical guarantee. It depends on correct measurement capture, comparable units, sufficient repetitions, and an experiment design that makes the numbers meaningful. Qualitative findings and mixed measurements still require human or agent interpretation.

This distinction reflects the project’s general philosophy: automate the structural and computable parts of project control while keeping judgment visible where the data does not justify certainty.

## The execution DAG turns current state into next work

The DAG engine provides several concrete controls:

- topological ordering keeps dependencies before dependent phases;
- actionable-phase selection requires dependencies to be complete;
- impact ordering prioritizes the most consequential currently available work;
- completed and deprioritized phases are excluded;
- repeated failed or inconclusive experiments can trigger a stagnation signal;
- dependency completion immediately changes what becomes available next.

The current implementation does not turn every evidence change into an opaque automatic workflow transition. Review and status changes remain explicit where project judgment is required. The design direction is controlled automation from inspectable state, not an autonomous planner whose rationale exists only in its prompt history.

## Decisions remain causal records

A decision without its reason is nearly useless to a later session. Project Manager records the decision text, rationale, project or experiment context, confidence/belief fields, and relationships to findings, constraints, or prior decisions.

When a decision changes, the system retains the earlier record and the relationship that changed its standing. That makes it possible to reconstruct both the historical logic and the present recommendation without rewriting project history.

## Retrieval returns a project neighborhood

A matching sentence is rarely enough. Text search can identify candidate objects, while graph traversal expands a finding, experiment, decision, hypothesis, constraint, or literature entry into its directly or indirectly connected neighborhood.

A search for “retrieval recall,” for example, can lead from the matching finding to the experiment that produced it, the hypothesis it tests, the decision it informed, and the phase that depends on the conclusion. The session receives a causal slice of the project rather than unrelated excerpts from several old notes.

## One domain core, three working surfaces

| Surface | Use |
|---|---|
| **CLI** | Fast terminal work, scripting, direct inspection, and repository-local workflows. |
| **MCP stdio** | Agent access to project orientation, node and edge operations, next-work selection, search, review, sessions, and statistics. |
| **Embedded web dashboard** | Human inspection of projects, phases, evidence, dependencies, contradictions, and structural health. |

The surfaces operate on the same SQLite-backed domain model and validation rules. MCP is not a separate agent-only memory store, and the dashboard does not maintain an independently authored project truth.

SQLite is intentional. It keeps a project portable, inspectable, backupable, and usable without deploying a network service. That tradeoff fits local and repository-scoped work; it does not claim to provide a multi-tenant collaboration database by itself.

## Implementation status

The implemented core includes:

- the typed store and migration history;
- project, phase, experiment, finding, decision, principle, hypothesis, constraint, literature, research, feedback, and session records;
- typed evidence edges and graph traversal;
- dependency-aware phase selection and stagnation checks;
- numeric measurement analysis;
- contradiction candidate detection and review support;
- temporal deltas, staleness and orphan review, search, and project context;
- CLI, MCP, and embedded web surfaces over the same database.

Active work is concentrated on richer causal-backbone traversal, stronger project-scoped context assembly, closer parity for newer operations across surfaces, and more explicit review-gate orchestration. Those are extensions of the operating model rather than a claim that every possible project-management workflow is already automated.

## Limitations

- Typed evidence preserves structure but cannot guarantee that a finding is correct, complete, or honestly recorded.
- The current contradiction candidate layer is heuristic and domain-tuned; semantic confirmation and user judgment remain necessary.
- Numeric consistency scoring is only as meaningful as the experiment design and captured measurements.
- Truth-maintenance rules make disagreement visible but do not constitute a complete formal belief-revision system.
- A local SQLite store is portable and simple but is not, by itself, a concurrent multi-user service.
- Next-action quality depends on accurate dependencies, impact values, status, and review discipline.
- Agents can still add redundant, vague, or poorly connected project objects; structural review reduces but does not eliminate that failure.

## Relationship to the other systems

- **Baton** can produce candidate findings, decisions, and run summaries. Project Manager decides what becomes durable project knowledge; it does not supervise Baton’s live workers.
- **HomeCloud** can execute experiments and agents. Project Manager records what those experiments mean; it does not schedule GPUs or own sandboxes.
- **Flip** can link product-native sources and community work into a research process. Project Manager does not replace Flip’s social content, identity, or authorization model.

## Source

The implementation, tests, CLI/MCP surfaces, and deeper reference material are available in the public [Project Manager repository](https://github.com/wahargis/project-manager).