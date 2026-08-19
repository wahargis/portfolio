# Portfolio Architecture and Review Guide

This repository documents four independent systems. They share engineering concerns, but they are not presented as modules of one deployed platform and no integration between them is assumed.

The purpose of this page is comparative: it shows which problem each project owns, what state it treats as authoritative, and where a reviewer can verify the design in source code.

## Responsibility map

| Project | Primary user problem | Authoritative state | Central control boundary | Representative implementation |
|---|---|---|---|---|
| **Flip** | A community needs live conversation, durable knowledge, and useful AI participation without losing authorship, privacy, or provenance. | Accounts, membership, rooms, messages, forum objects, synthesis runs, AI activity, citations, and artifacts. | The product decides which actor may read or act, which context an AI participant receives, and what durable effect is committed. | Phoenix contexts, PostgreSQL/Ecto, Oban workflows, Channels/LiveView, client synchronization. |
| **Project Manager** | Long-running technical work needs a durable explanation of evidence, decisions, contradictions, dependencies, and handoffs. | Typed project nodes and edges, phase state, analysis results, and retrieval/session records. | The evidence graph explains what is known; the execution DAG explains which dependency-satisfied phase can proceed. | Shared Rust core, SQLite store, graph traversal, DAG scheduling, analysis, CLI/MCP/web surfaces. |
| **Baton** | Parallel coding-agent work needs explicit intent, scope, interaction, workspace ownership, verification, and adoption authority. | Versioned goals and plans, routes, sessions, events, interactions, workspaces, evidence, verification, and result records. | Workers may implement; Baton retains authority over admission, scope, lifecycle, verification, and publication. | Node.js control plane, persistent-session adapters, waves, real Git worktrees, verification sandboxes, result export, CLI/MCP/web. |
| **HomeCloud** | Local AI hardware needs to become a dependable shared runtime rather than a set of manually managed model processes. | Model instances, capacity, GPU claims, queues, agent tasks, checkpoints, research state, application records, and health. | The runtime decides which inference path and hardware capacity are available, how work is prioritized, and where tools execute. | Elixir/OTP supervision, Phoenix/Ash/PostgreSQL, model adapters, GPU scheduling, sandboxes, recoverable agent execution. |

## The four systems answer different questions

### Flip: what may happen inside a community product?

Flip owns social and product semantics. It determines identity, membership, audience, message and forum state, AI triggers, eligible context, product tools, provenance, and publication.

A model endpoint can fail without changing ordinary community authority. AI capability is one part of the product, not an alternate backend with independent access to community data.

### Project Manager: what does the project know, and what work is ready?

Project Manager stores typed evidence and execution state across sessions. Its evidence graph can represent support, contradiction, supersession, derivation, testing, and dependency. Its DAG can order project phases and recommend dependency-satisfied work.

The current implementation keeps contradiction/confidence analysis and scheduling adjacent but distinct. Review signals can inform a person or agent changing the plan; they do not automatically act as hard scheduler gates.

### Baton: how is delegated software work controlled?

Baton starts from an approved objective and plan, selects a compatible harness/model/effort route, allocates isolated workspaces, supervises persistent sessions, handles structured human interactions, and verifies captured artifacts.

The model or coding harness owns its native reasoning loop. Baton owns the surrounding delivery contract and does not treat a worker’s completion message as acceptance evidence.

### HomeCloud: how is finite AI capacity operated?

HomeCloud manages model services and accelerator capacity for interactive requests, agents, research, and background work. It exposes capacity through supervised pools, claims, priorities, health, and routing.

Agent work is executed through an application runtime with context, tools, sandboxes, loop controls, checkpoints, and persistent results. The platform is local-first but can use remote providers where policy and capability allow.

## Shared engineering themes

### State is modeled at the level required for recovery

Each project stores more than a transcript or log:

- Flip stores product objects, audience, source relationships, workflow state, and AI activity.
- Project Manager stores typed evidence, decisions, graph relationships, and phase state.
- Baton stores approved intent, route, session, interaction, repository ownership, and verification evidence.
- HomeCloud stores model capacity, claims, task state, checkpoints, and application records.

The relevant question is not whether data is durable in the abstract. It is whether the system has enough state to continue correctly after the originating request, model context, provider process, or operator session has ended.

### Capability does not imply authority

Across the portfolio, a component may be technically capable of an action while still lacking authority to perform or finalize it:

- A Flip AI participant can call only capabilities admitted for its product surface and actor scope.
- A Project Manager client can query or update only the typed project operations exposed by the shared core.
- A Baton worker may have a shell, but its repository scope and result adoption remain controller-owned.
- A HomeCloud agent can execute tools only through its assigned runtime and workspace.

### Provenance is operational data

Provenance is not presented as a decorative citation field. It affects behavior:

- Flip uses origin relationships to constrain access to synthesized forum content.
- Project Manager uses typed edges to preserve why a conclusion or decision exists.
- Baton ties a result to an approved plan, exact workspace, run lineage, and verification.
- HomeCloud ties work to model/infrastructure state, agent lifecycle, checkpoints, and stored results.

### Interfaces share semantics

CLI, MCP, web, native, connector, and provider-specific surfaces should call a common application model. A new interface should not become a weaker parallel implementation of authorization, state transition, or failure behavior.

### Non-determinism is bounded by deterministic systems

Models contribute interpretation, planning, retrieval selection, and composition. Code retains control of identity, scope, schemas, lifecycle, persistence, resource ownership, and verification.

The portfolio therefore evaluates more than response quality. It asks whether the surrounding system makes AI behavior observable, interruptible, recoverable, and safe to integrate.

## Important project boundaries

| Boundary | What the portfolio claims | What it does not claim |
|---|---|---|
| **Flip and model infrastructure** | Flip can use provider-compatible local or hosted inference. | Flip’s product authority depends on HomeCloud or any one provider. |
| **Project Manager analysis and DAG** | Contradiction and confidence tools provide review evidence next to deterministic phase scheduling. | A belief-state change automatically blocks or rewrites the DAG. |
| **Baton and coding harnesses** | Baton coordinates persistent native harness sessions through capability-aware adapters. | Every harness exposes identical controls, evidence, or reliability. |
| **HomeCloud and local hardware** | HomeCloud schedules heterogeneous local capacity and can route to remote providers. | One private hardware topology defines the product or every deployment. |
| **Cross-project use** | The projects could exchange standards-based data or services where useful. | They are currently one integrated platform or require one another to operate. |

## Supporting repositories

The flagship pages cover the systems above. Related repositories support their product or deployment surfaces:

- Flip’s React/TypeScript native-client work belongs to the Flip product architecture.
- HomeCloud tools and adapters connect external applications to HomeCloud services.
- These supporting repositories are not presented as additional flagship systems merely to increase project count.

## Documentation standard

Portfolio documentation should let a new reader answer, in order:

1. What is the product or operational capability?
2. Who uses it, and what problem does it solve?
3. What does a representative workflow look like?
4. Which state and authority boundaries make that workflow reliable?
5. Which source modules implement the important claims?
6. What remains deliberately out of scope or not yet coupled?

It should avoid:

- opening with internal subsystem names before establishing the product;
- listing frameworks or modules without explaining their consequence;
- implying integration because two projects could exchange data;
- converting issue history, deployment choreography, or private configuration into portfolio narrative;
- presenting planned automation as shipped behavior;
- using unsupported scale, novelty, maturity, or quality claims.

[← Back to portfolio](README.md)
