# Flip Diagram Index

These diagrams document product and agent execution flows. They show state ownership, authorization, persistence, effects, failure, and recovery rather than only naming components.

| Diagram | System behavior |
|---|---|
| [Product runtime](product-runtime.svg) | One direct AI request from product trigger through actor scope, context, capability selection, model and tool execution, committed effects, durable state, and client delivery. |
| [Platform execution](platform-execution.svg) | Direct and asynchronous agent execution, including pending artifacts and deduplicated continuation. |
| [System context](system-context.svg) | Members, operators, clients, workflows, product domains, agent services, durable state, and external providers. |
| [Service and container map](service-container-map.svg) | Phoenix entry points, product contexts, agent services, PostgreSQL authority, workers, realtime delivery, and provider adapters. |
| [Agent execution sequence](agent-execution-sequence.svg) | Turn admission, context, route and tool admission, working rounds, terminal validation, product commit, and continuation. |
| [Governed tool execution](governed-tool-execution.svg) | Capability lookup, schema validation, trusted scope, authorization, read-only results, committed effects, pending artifacts, and refusals. |
| [Retrieval and citation flow](retrieval-source-citation-flow.svg) | Internal retrieval, external discovery, direct source reads, evidence records, citation validation, and incomplete-evidence handling. |
| [Conversation curation](synthesis-pipeline.svg) | Source selection, curation run, planning, validation, forum publication, provenance, access, partial failure, and correction. |
| [Client synchronization](client-synchronization.svg) | Optimistic commands, canonical commits, durable synchronization, realtime delivery, deduplication, offline recovery, and access revocation. |
| [Model routing and audit](model-routing-audit.svg) | Workload requirements, route selection, provider adaptation, local or hosted execution, activity records, fallback, evaluation, and visible failure. |
| [Product and synthetic environments](deployment-topology.svg) | Shared contracts with separate data, credentials, queues, storage, callbacks, and authority. |
| [Testing and evaluation](ci-quality-gates.svg) | Deterministic product checks, agent and tool fixtures, background recovery, client convergence, security, route evaluation, and operational evidence. |

Mermaid source is stored beside the rendered SVGs. Public diagrams omit product data, credentials, provider keys, private host configuration, prompt and persona state, and administrative access.
