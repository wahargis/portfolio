# 08 — Production and Demo Topology

Flip maintains two live targets with a separation boundary.

| Concern | Production | Technical demo |
|---|---|---|
| Host | `flip.engineering` | `flip.tech-demo.dev` |
| Data | Authorized production | Versioned synthetic seed data |
| Database/storage | Production resources | Separate resources |
| Credentials | Production routing | Separate capped credentials |
| Accounts | Real users | Stable demo accounts |
| Reset | Normal retention | Reproducible reset |
| Client target | Default production | Explicit demo profile |
| UI | Canonical product | Persistent technical-demo banner |

Production deployments are workflow-gated: a deploy workflow runs in the production environment on a self-hosted runner, serializes deploys with `flock`, fast-forwards the canonical checkout, runs drift checks against `https://flip.engineering/api/version`, and invokes a guarded host redeploy.

Demo deployments use `bin/deploy-beta`, which brings up the stack and a Cloudflare Tunnel and verifies `https://flip.tech-demo.dev/api/health/ready`. `docker-compose.yml` includes both hosts in `CORS_ORIGINS` and `CHECK_ORIGIN`.

Both hosts expose health and version endpoints for operational visibility, and both are live for review.

<img src="../diagrams/deployment-topology.svg" alt="Production and demo deployment topology" width="760" />
