# Diagrams

This directory contains Mermaid sources (`.mmd`) and rendered SVG/PNG assets for the Flip public architecture overview.

## Diagram index

| Source | Rendered SVG | Rendered PNG | View |
|---|---|---|---|
| `system-context.mmd` | `system-context.svg` | `system-context.png` | Users, web/desktop clients, Phoenix, data, AI, external sources |
| `service-container-map.mmd` | `service-container-map.svg` | `service-container-map.png` | Deployment services, client paths, external services, hosts |
| `agent-execution-sequence.mmd` | `agent-execution-sequence.svg` | `agent-execution-sequence.png` | Bounded agent run from Oban job to terminal result |
| `retrieval-source-citation-flow.mmd` | `retrieval-source-citation-flow.svg` | `retrieval-source-citation-flow.png` | Source discovery through citations and ledger |
| `synthesis-pipeline.mmd` | `synthesis-pipeline.svg` | `synthesis-pipeline.png` | Chat trigger through persisted synthesis with provenance |
| `client-synchronization.mmd` | `client-synchronization.svg` | `client-synchronization.png` | Electric shape sync and in-memory client behavior |
| `deployment-topology.mmd` | `deployment-topology.svg` | `deployment-topology.png` | Production deploy path and demo tunnel path |
| `governed-tool-execution.mmd` | `governed-tool-execution.svg` | `governed-tool-execution.png` | Tool allowlist, authz scope, URL guard, isolated dispatch |
| `model-routing-audit.mmd` | `model-routing-audit.svg` | `model-routing-audit.png` | LLM provider routing, video routing, call audit |

## How to render

Each diagram is rendered with mermaid-cli:

```bash
npx -y @mermaid-js/mermaid-cli -i diagrams/system-context.mmd -o diagrams/system-context.svg -b white
npx -y @mermaid-js/mermaid-cli -i diagrams/system-context.mmd -o diagrams/system-context.png -b white -s 2
```

Replace the input and output names for the other diagrams.
