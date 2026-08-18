# Portfolio architecture

The portfolio contains four systems at adjacent layers of AI-assisted work:

1. **Flip** owns a human-facing collaboration product and the semantics of AI participation inside it.
2. **Project Manager** owns long-horizon project state: evidence, hypotheses, decisions, constraints, contradiction, and next work.
3. **Baton** owns cross-harness fleet orchestration: planning, routing, persistent workers, interaction, shared context, workflows, capabilities, and run lifecycle.
4. **HomeCloud** owns self-hosted model and execution infrastructure: inference capacity, GPU scheduling, sandboxes, context/memory services, and runtime recovery.

The projects are related, but they were not designed as modules of one master platform. Their boundaries are useful precisely because each can be deployed, understood, and evolved independently.

## Responsibility map

| Concern | Flip | Project Manager | Baton | HomeCloud |
|---|---:|---:|---:|---:|
| Human-facing community product | **Primary** | — | — | — |
| Chat, forum, identity, moderation, product permissions | **Primary** | — | — | — |
| AI participation inside a product context | **Primary** | Records related knowledge | Can modify source under a run | Hosts inference/execution |
| Long-horizon project evidence and belief revision | Links product sources | **Primary** | Promotes selected run knowledge | Produces experiment results |
| Project phases and computed next work | — | **Primary** | Executes planned work | Supplies capacity |
| Cross-vendor full-session harness orchestration | — | — | **Primary** | Can host local seats/endpoints |
| Harness/model/effort selection | Selects inference route | — | **Primary for coding fleets** | Selects local model placement |
| Multi-worker plans, waves, workflows, and hierarchy | — | — | **Primary** | Runs lower-level tasks |
| Worker communication, attention, interruption, and steering | Product AI interactions | — | **Primary** | Agent runtime events only |
| Fleet-shared context, scratch, memory, and capabilities | Product context | Project graph | **Primary** | Runtime memory/tools |
| Local model serving and GPU allocation | Consumes | Consumes | Consumes when local | **Primary** |
| Sandboxed autonomous execution and checkpoints | Product jobs | Session records | Harness/worktree lifecycle | **Primary runtime** |
| Source verification and integration | Product CI | Records decisions | Run-level verification/review/integration | Supplies evaluators/sandboxes |

“Primary” means the system defines the domain model and public contract for that concern. A neighboring system may provide data or capacity without redefining those semantics.

## Composition model

```text
Human communities
       |
       v
     Flip  <---------------------->  hosted or HomeCloud model endpoints
       |
       | durable product artifacts / selected findings
       v
Project Manager <-----------------> Baton
       ^                    run findings, decisions, handoffs
       |                              |
       | experiment results           | local seats / model / sandbox capacity
       +-------------------------- HomeCloud
```

The arrows are optional integration contracts. They do not imply that every deployment wires all four systems together.

## Flip and HomeCloud

Flip consumes provider-compatible inference. HomeCloud can supply a local endpoint without changing Flip’s chat, forum, membership, authorship, tool-scope, artifact, or provenance model.

The contract is asymmetric:

- Flip decides product context, permissions, admitted tools, response persistence, and user-visible behavior.
- HomeCloud decides local endpoint health, model placement, slot scheduling, and GPU lifecycle.
- A local inference failure degrades an AI capability; it does not become a social-product state transition.

## Baton and HomeCloud

Baton’s primary abstraction is a **full coding harness session**, not a GPU process. HomeCloud may provide local models, execution sandboxes, or host capacity, while Baton remains responsible for:

- which harness, exact model, and effort route a Run uses;
- Goal/Plan compilation and dependency scheduling;
- persistent worker sessions and their adapter capabilities;
- messages, questions, attention, turn checkpoints, interruption, and steering;
- shared fleet context, artifacts, knowledge, and capability services;
- Run result, review, evidence, adoption, integration, and cleanup.

HomeCloud does not need to understand a Baton workflow, and Baton does not need to know HomeCloud’s GPU allocation internals.

## Baton and Project Manager

Baton and Project Manager both retain durable state, but at different tempos and for different purposes.

Baton’s operational and coordinative state answers:

- What is this fleet doing now?
- Which worker/session owns which task and artifact?
- What is the Run waiting on?
- What context or result should another member receive?
- How can the workflow recover or continue?

Project Manager answers:

- What does the project currently believe?
- Which experiment or finding supports a decision?
- What contradicts or supersedes that belief?
- Which phase should happen next across sessions?

Selected Baton findings, decisions, constraints, and run scorecards can be promoted or exported into longer-horizon project state. Raw fleet telemetry is not automatically project knowledge, and Project Manager does not become Baton's process supervisor.

## Flip and Project Manager

Flip preserves product-native provenance: messages, threads, replies, citations, artifacts, participant feedback, and curation history. Project Manager can model a more explicit research process when a community workflow needs experiments, hypotheses, decisions, principles, or constraints. It does not replace Flip’s social content model.

## Cross-cutting engineering practices

### Preserve the useful native system

- Baton drives vendor-native full harnesses rather than replacing them with raw API calls.
- Flip keeps chat and forum as distinct interaction models instead of flattening both into an AI transcript.
- HomeCloud exposes model capacity through supervised endpoints and slots rather than hard-coding hardware assumptions into applications.
- Project Manager preserves typed evidence relationships rather than reducing research to generated summaries.

### Store the state needed by the domain

Durable state is not synonymous with chat history. Each system records the identities, relationships, lifecycle, or recovery information needed to continue correctly after the originating model context is gone.

### Share semantics across interfaces

Where CLI, MCP, web, native, or embedded surfaces coexist, they should call one domain/application model and expose compatible state and refusal semantics. A new interface is not permission to create a weaker parallel implementation.

### Use supporting machinery in service of the product

Reliability mechanisms should explain how a capability works, not replace its purpose in the documentation. Verification is a feature of Baton’s fleet driver; queue isolation is a feature of Flip’s product workflows; GPU claims are a feature of HomeCloud’s runtime.

### Prefer progressive disclosure

A reviewer should be able to understand the problem and system shape before encountering exact commands, event schemas, provider quirks, or issue history. Deep technical detail remains available where it explains a real capability or tradeoff.

## Supporting systems

### Flip client

The React/TypeScript, Tauri, Capacitor, and Electric client work belongs to Flip’s product architecture. It is not presented as an additional flagship.

### HomeCloud tools

Dispatch adapters, assistant plugins, and collaboration tooling connect external harnesses to HomeCloud and informed Baton’s orchestration work. Baton remains deployment-neutral and has no required HomeCloud runtime dependency.

## Documentation contract

The portfolio should be detailed about:

- project aims and user-visible capability;
- architecture and data boundaries;
- state models and execution lifecycles;
- cross-harness or cross-client interaction;
- context, memory, and recovery;
- implemented versus active versus research capability;
- meaningful engineering tradeoffs.

It should avoid:

- recasting a supporting subsystem as the product;
- repetitive deployment or reviewer choreography;
- issue-number inventories where a stable capability explanation is possible;
- source-derived secrets, production data, and host-specific configuration;
- unsupported metrics, novelty claims, or maturity labels.
