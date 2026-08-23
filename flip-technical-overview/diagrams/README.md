# Flip diagram index

The diagrams cover product boundaries, agent execution, data flow, and failure handling. SVG is the primary format. Mermaid source is stored beside the rendered diagram where available.

| Diagram | Content |
|---|---|
| [Product-integrated agent execution](platform-execution.svg) | Admission from a product event, actor and object scope, context, route and capability selection, model and tool execution, durable effects, asynchronous continuation, and client updates. |
| [System context](system-context.svg) | Product actors, clients, external providers, and the limits of Flip's authority. |
| [Service and container map](service-container-map.svg) | Phoenix application contexts, jobs, data, realtime delivery, clients, and provider adapters. |
| [AI reply lifecycle](agent-execution-sequence.svg) | The sequence from an explicit trigger through context, tools, terminal validation, and reply persistence. |
| [Governed tool execution](governed-tool-execution.svg) | Capability admission, trusted scope, authorization, tool execution, effects, and result handling. |
| [Retrieval, source discovery, and citation](retrieval-source-citation-flow.svg) | External discovery, direct source reads, source records, citation validation, and terminal use. |
| [Chat-to-forum curation](synthesis-pipeline.svg) | Source selection, planning, validated forum mutation, participant and message provenance, and linkback. |
| [Web and native client state](client-synchronization.svg) | Server commands, PostgreSQL authority, durable synchronization, ephemeral channels, and client reconciliation. |
| [Model routing and audit](model-routing-audit.svg) | Product-owned route policy, provider adapters, activity records, and failure handling. |
| [Product and synthetic environment](deployment-topology.svg) | Shared application contracts with separate data, credentials, queues, and authority. |
| [Testing and evaluation](ci-quality-gates.svg) | Deterministic product tests, client and integration tests, security tests, route evaluation, and operational evidence. |

These diagrams omit credentials, private host configuration, product data, provider keys, and administrative access. They include the state and control boundaries needed to understand the system.
