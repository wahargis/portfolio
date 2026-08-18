# Portfolio architecture

The projects in this repository are related by a common systems question: **where should authority live when AI is useful for reasoning but unreliable as a source of truth?**

They answer that question at different layers.

Flip applies it inside a social product. Project Manager applies it to long-running knowledge and decision state. Baton applies it to coding-agent execution. HomeCloud applies it to compute and runtime infrastructure. The projects line up conceptually, but they were not designed to become one all-encompassing AI platform.

That distinction matters because a portfolio can easily make independent systems look less credible by forcing them into an artificial taxonomy. The useful relationship here is narrower: each project gives deterministic software responsibility for the part of the problem where ambiguity would be dangerous, while leaving models to operate where semantic judgment is valuable.

## Where the boundaries sit

### Flip owns product meaning

Flip decides who a user is, what they may see, which community a request belongs to, whether content is human- or AI-authored, and how a durable artifact relates back to the discussion that produced it.

If Flip uses a local model served by HomeCloud, that does not transfer product authority to the inference layer. The model endpoint can disappear and the social product should remain internally coherent. Conversely, HomeCloud does not need to know what a forum thread means in order to schedule the inference request correctly.

### Project Manager owns project belief

Project Manager is concerned with what a project currently believes and why. It can ingest a finding produced by a Baton-controlled coding campaign or an experiment run on HomeCloud, but raw execution telemetry is not automatically project knowledge.

That boundary protects both sides. Baton remains free to record detailed lifecycle evidence without turning into a research notebook. Project Manager can preserve causality and contradiction without becoming a process supervisor.

### Baton owns execution authority

Baton is responsible for whether a worker is alive, what source it owns, which commands target the current incarnation, whether a result can be reproduced, and whether that result is eligible for adoption.

It can use HomeCloud for local models or sandboxes, but compute availability does not imply source-control authority. Likewise, a worker may produce a technically interesting finding for Project Manager, but that does not make the worker’s self-report authoritative.

### HomeCloud owns compute and runtime concerns

HomeCloud decides how local capacity is represented, scheduled, supervised, and recovered. It is deliberately below the semantic layer of the applications that consume it.

That lets the hardware and model mix evolve without forcing Flip, Baton, or Project Manager to adopt HomeCloud’s internal abstractions. The contract is closer to infrastructure: provide healthy, observable execution capacity and keep failure local.

## The recurring design pattern

Across all four projects, the same judgment appears in different forms.

**A model response is a proposal, not a state transition.** Product permissions, source adoption, project belief, and process lifecycle are all committed by code that has an explicit representation of the relevant state.

**Durable state matters more than transcript memory.** A useful system needs to retain causality, provenance, ownership, or recovery information in forms that survive the context window that produced them.

**Verification should be independent of the worker being verified.** That can mean preserving source-message provenance in Flip, contradiction relationships in Project Manager, fresh-worktree checks in Baton, or deterministic evaluators in HomeCloud research workloads.

**Isolation comes before autonomy.** Agents and model-serving processes are more useful when failures have bounded ownership. Worktrees, sandboxes, schedulable slots, scoped retrieval, and explicit lifecycle state are all manifestations of that principle.

## Why the projects remain separate

There is an obvious temptation to turn this into a single diagram where Flip feeds Project Manager, Baton runs everything, and HomeCloud sits underneath as the universal substrate. That would be architecturally neat and misleading.

The systems solve different problems and should continue to be understandable on their own. Their integration points are useful precisely because they are narrow:

- Flip can consume a provider-compatible model endpoint without depending on HomeCloud.
- Baton can control remote or local coding harnesses without depending on Project Manager.
- Project Manager can record evidence produced by people, scripts, or agents without caring which executor generated it.
- HomeCloud can serve unrelated workloads without knowing the product semantics above it.

The portfolio therefore presents these as **independent systems with compatible boundaries**, not as modules of an invented master platform.

## Documentation standard

The case studies should explain enough implementation detail to make the architecture credible, but detail should earn its place.

A mechanism belongs in the portfolio when it explains a design decision, a difficult failure mode, a trust boundary, or an unusual product capability. A list of every tool, enum, worker, protocol, provider, or record type does not demonstrate depth by itself; it usually hides the engineering judgment the reader actually needs to see.

That is the standard used for the project pages in this repository: problem, decision, mechanism, consequence, limitation.
