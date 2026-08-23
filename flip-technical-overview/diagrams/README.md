# Flip Diagram Index

The diagrams isolate distinct parts of the product and runtime. SVG files are the rendered documents; Mermaid sources are retained beside diagrams where the source format is useful.

| Diagram | System detail |
|---|---|
| **[Product and AI runtime](product-runtime.svg)** | Community surfaces, shared domains, the complete AI participant turn, conversation curation, and durable/realtime infrastructure. |
| [System context](system-context.svg) | Members, operators, clients, providers, and systems outside Flip's product authority. |
| [Service and container map](service-container-map.svg) | Phoenix contexts, background work, PostgreSQL, synchronization, realtime delivery, and external capabilities. |
| [AI reply lifecycle](agent-execution-sequence.svg) | Trigger admission, context construction, model and tool rounds, evidence, terminal composition, and message publication. |
| [Governed tool execution](governed-tool-execution.svg) | Capability selection, trusted actor scope, argument validation, effect authority, execution, and audit. |
| [Retrieval and citation flow](retrieval-source-citation-flow.svg) | Internal, external, and document sources becoming durable evidence and citations. |
| [Chat-to-forum synthesis](synthesis-pipeline.svg) | Source selection, synthesis state, forum publication, lineage, and feedback. |
| [Client synchronization](client-synchronization.svg) | Commands, optimistic state, server transactions, Electric delivery, and ephemeral Channels. |
| [Model routing and audit](model-routing-audit.svg) | Product route policy, provider adapters, failure classification, activity, and usage records. |
| [Deployment topology](deployment-topology.svg) | Product and synthetic environments sharing application architecture without sharing data or authority. |
| [CI and quality gates](ci-quality-gates.svg) | Deterministic domain tests, client checks, provider evaluation, security checks, and release evidence. |

The primary diagram is embedded in the [Flip case study](../README.md). The remaining diagrams accompany the consolidated technical documentation:

- [Product and domain model](../docs/00-product-and-domain.md)
- [System and data architecture](../docs/01-system-and-data-architecture.md)
- [AI participant runtime](../docs/02-ai-participant-runtime.md)
- [Retrieval, tools, and artifacts](../docs/03-retrieval-tools-and-artifacts.md)
- [Clients and deployment](../docs/04-clients-and-deployment.md)
- [Testing, operations, and current status](../docs/05-testing-operations-and-status.md)

The diagrams omit production data, credentials, private hostnames, proprietary prompts and personas, exact abuse thresholds, and operational details that are not necessary to explain the architecture.
