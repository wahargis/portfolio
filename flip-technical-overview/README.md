# Flip — Technical Overview

Flip is a real-time chat and durable forum platform with configurable AI participants. It combines live discussion, persistent decision records, and governed AI assistance so teams can talk quickly and verify what was decided.

## Live product

Flip runs in two public environments: the production site at <https://flip.engineering> and a synthetic technical demo at <https://flip.tech-demo.dev>. The demo uses versioned synthetic data, stable demo accounts, separate credentials, and a reproducible reset path; it never shares production data or credentials.

<figure>
  <img src="assets/flip.engineering.png" alt="Flip production site screenshot" width="760" />
  <figcaption>Flip production: chat and forum surfaces on <a href="https://flip.engineering">flip.engineering</a>.</figcaption>
</figure>

<figure>
  <img src="assets/flip.tech-demo.dev.png" alt="Flip technical demo screenshot" width="760" />
  <figcaption>Flip technical demo: a separate, synthetic environment on <a href="https://flip.tech-demo.dev">flip.tech-demo.dev</a>.</figcaption>
</figure>

## Why chat and forums are combined

Chat is immediate; forums are permanent. Chat alone loses decisions in the message stream, and forums alone are too slow for live exchange. Flip keeps both in one workspace: conversations happen in real time, and when a conversation reaches a decision, it can be synthesized into a forum thread that stays linked to the exact messages that produced it.

## How the AI participant works

Each room can configure one or more AI participants with a persona, a model, and tool budgets. When synthesis is enabled, a background run starts a bounded agent: it gathers information through governed tools under round, time, and token limits; then it persists the result as a forum thread or reply with quote-verified citations and a reader-facing source ledger.

## Main architecture

<img src="diagrams/system-context.svg" alt="Flip system context diagram" width="760" />

<img src="diagrams/service-container-map.svg" alt="Flip service and container architecture diagram" width="760" />

## Representative capabilities

1. **Hybrid chat + forums** — real-time rooms and threads share one product surface, with a web client and a native desktop client.
2. **Bounded agent runtime** — AI work runs under explicit caps on rounds, time, tokens, and tool execution, with isolated per-tool dispatch.
3. **Citation and provenance** — AI-minted, quote-verified web/PDF citations, forum source tracking, and a reader-facing source ledger.
4. **Real-time desktop sync** — a native desktop client receives live updates while authoritative writes stay with the server.

## Product walkthrough

Open the technical demo at <https://flip.tech-demo.dev>, sign in with a stable demo account, and try the scenarios in [demo/README.md](demo/README.md): ask a question that requires external search, watch the AI participant cite sources, then synthesize a chat exchange into a forum thread and inspect its provenance.

## Deeper architecture pages

1. [docs/00 — Executive Overview](docs/00-executive-overview.md)
2. [docs/01 — Product and Problem](docs/01-product-and-problem.md)
3. [docs/02 — System Architecture](docs/02-system-architecture.md)
4. [docs/03 — Agent Runtime](docs/03-agent-runtime.md)
5. [docs/04 — Retrieval, Search, and Tools](docs/04-retrieval-search-and-tools.md)
6. [docs/05 — Synthesis and Provenance](docs/05-synthesis-and-provenance.md)
7. [docs/06 — Data, Realtime, and Clients](docs/06-data-realtime-and-clients.md)
8. [docs/07 — Model Routing and Inference](docs/07-model-routing-and-inference.md)
9. [docs/08 — Production and Demo Topology](docs/08-production-and-demo-topology.md)
10. [docs/09 — Evaluation, Testing, and Operations](docs/09-evaluation-testing-and-operations.md)
11. [docs/10 — Architecture Decisions](docs/10-architecture-decisions.md)
12. [docs/11 — Roadmap and Known Limitations](docs/11-roadmap-and-known-limitations.md)

Rendered diagrams live in [diagrams/](diagrams/README.md); public design principles live in [adr/](adr/README.md).
