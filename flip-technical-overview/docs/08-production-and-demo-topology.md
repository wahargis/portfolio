# 08 — Production and Demo Topology

Flip maintains two live targets with a clear separation boundary: a production environment and a synthetic technical demo.

<img src="../diagrams/deployment-topology.svg" alt="Production and demo deployment topology" width="760" />

## Separation boundary

| Concern | Production | Technical demo |
|---|---|---|
| Host | `flip.engineering` | `flip.tech-demo.dev` |
| Data | Authorized production | Versioned synthetic seed data |
| Resources | Production resources | Separate resources |
| Credentials | Production routing | Separate capped credentials |
| Accounts | Real users | Stable demo accounts |
| Reset | Normal retention | Reproducible reset |
| Client target | Default production | Explicit demo profile |
| UI | Canonical product | Persistent technical-demo banner |

## Deployment approach

Production deployments are workflow-gated: changes pass automated checks before reaching the live host, and deployments are serialized to avoid conflicts. The demo is deployed as a separate environment with its own resources, seeded from synthetic data only, and can be reset reproducibly.

## Operational visibility

Both hosts expose health and version information for operational visibility, but the exact endpoints, scripts, and deployment mechanics are not part of the public architecture. What matters publicly is that production and demo remain isolated and reviewable.
