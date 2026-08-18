# Diagram index

The diagrams answer architectural questions. SVG is the canonical reviewer format and Mermaid source is retained beside it. Raster previews are generated during review but are not versioned, preventing stale PNG copies from diverging from the source diagram.

| Diagram | Question answered |
|---|---|
| [System context](system-context.svg) | Who interacts with Flip and which external systems are outside its authority? |
| [Service/container map](service-container-map.svg) | How do clients, product contexts, asynchronous jobs, data, sync, and providers fit together? |
| [AI reply lifecycle](agent-execution-sequence.svg) | How does a product trigger become one bounded, durable AI reply? |
| [Governed tool execution](governed-tool-execution.svg) | Where are catalog admission, trusted scope, isolation, authorization, and effects enforced? |
| [Retrieval, source discovery, and citation](retrieval-source-citation-flow.svg) | How does a discovered source become verified evidence in a user-facing reply? |
| [Chat-to-forum synthesis](synthesis-pipeline.svg) | How is human conversation organized into durable forum structure with source provenance? |
| [Web/native realtime flow](client-synchronization.svg) | How do HTTP commands, durable synchronization, and ephemeral channels divide responsibility? |
| [Model routing and audit](model-routing-audit.svg) | How is model/provider choice isolated from product semantics and failure handling? |
| [Product/synthetic inference topology](deployment-topology.svg) | How do separate environments share architecture without sharing authority or data? |
| [CI and quality gates](ci-quality-gates.svg) | Which deterministic, client, security, and evaluation evidence supports release confidence? |

## Reading order

1. System context
2. Service/container map
3. AI reply lifecycle
4. Governed tool execution
5. Chat-to-forum synthesis
6. Web/native realtime flow

The remaining diagrams support specialized review.

## Scope

These are conceptual, implementation-informed diagrams. They intentionally omit private hostnames, credentials, provider keys, exact abuse thresholds, private data, and low-value operational detail.
