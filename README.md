# AI Systems & Agent Engineering Portfolio

Four implemented systems spanning AI product architecture, agent execution and orchestration, durable research state, data modeling, and local inference operations. Each project begins with a concrete product or operating problem and follows it through the state model, execution model, failure handling, and implementation that make the system work.

## [Flip](flip-technical-overview/)

**Real-time community platform and in-product AI runtime**

Flip combines live chat, threaded forums, media, and explicit AI participation in one product. Conversation remains immediate and social, while selected discussion can move through a durable synthesis workflow into searchable forum knowledge without losing authorship, source context, or audience restrictions.

The engineering work extends well beyond model integration. Flip includes a product-owned agent runtime for trigger detection, actor and room authorization, bounded context construction, capability admission, multi-round tool execution, provider routing, citations, artifacts, continuation of long-running media work, and durable activity records. Phoenix, PostgreSQL, Oban, PubSub, Channels, LiveView, and Electric-based client synchronization carry the resulting state across web and native clients. The public case study includes product and synthetic-environment screens plus a complete runtime and curation architecture.

**[Project case study](flip-technical-overview/)** · **[Technical documentation](flip-technical-overview/docs/00-product-and-domain.md)** · **[Live product](https://flip.engineering)** · **[Synthetic technical environment](https://flip.tech-demo.dev)**

## [Project Manager](project-manager/)

**Local-first research operations and durable project state for people and agents**

Project Manager manages work that outlives a task list, chat session, or original plan. Projects, phases, experiments, findings, hypotheses, decisions, literature, constraints, principles, feedback, and sessions are stored as typed operational state rather than reconstructed from notes after the fact.

Its agent-facing design combines guarded MCP workflows, causal evidence requirements, dependency-aware execution, temporal context reconstruction, project-scoped retrieval, truth-maintenance operations, and explicit review and repair surfaces. A deterministic phase DAG selects dependency-satisfied work; contradiction and confidence analysis remain separate review inputs rather than opaque scheduling decisions. The case study renders the public `atlas` quickstart as one concrete phase DAG and evidence-to-decision record.

**[Project case study](project-manager/)** · **[Public source](https://github.com/wahargis/project-manager)**

## [Baton](baton/)

**Run-centric orchestration for full coding-harness sessions across providers**

Baton coordinates persistent vendor-native coding harnesses as a fleet. An objective can be compiled into a plan or declarative workflow, routed onto exact harness, model, and effort combinations, and executed through parallel waves with visible member state, structured interaction, steering, shared context, recovery, and result collection.

The system unifies embedded, resident, CLI, and MCP control surfaces over one run model. Provider adapters preserve native session capabilities; the coordinator maintains event and lifecycle state; workers receive isolated Git worktrees; collaboration state carries questions, replies, scratchpads, knowledge, and briefing packs; and harvested results can proceed through verification, review, adoption, and integration. The case study includes a recorded two-worker wave in which both sessions completed but a missing harvested result correctly produced an incomplete workflow verdict.

**[Project case study](baton/)** · **[Public source](https://github.com/Flip-Engineering/baton)**

## [HomeCloud](homecloud/)

**Local-first AI operations platform and recoverable agent runtime**

HomeCloud turns heterogeneous owned compute into a supervised application platform for inference, agents, research, documents, connectors, browser work, and multimodal generation. Applications request capabilities; the runtime resolves model and backend policy, admits work against actual capacity, manages GPU and model-service ownership, and returns durable results to product state.

The implementation combines OTP supervision, priority-aware model-instance pools, inference routing, GPU claims and workload scheduling, containerized agent workspaces, dynamic tool selection, loop control, checkpoint recovery, telemetry, and long-running research execution. The dated reference deployment shows the four-V100 32 GB local pool, separate A100 Drive 32 GB accelerator, hosted-provider fallbacks, durable application state, and Flip hosted as an application workload.

**[Project case study](homecloud/)**

## Architecture across the projects

The projects are independently deployable and do not assume one another. Their boundaries and complementary engineering coverage are documented in **[Systems architecture across the portfolio](PORTFOLIO_ARCHITECTURE.md)**.

Project Manager and Baton publish their implementation repositories. Flip and HomeCloud are represented through the public case studies, diagrams, and selected implementation evidence in this repository because their implementation repositories are private.
