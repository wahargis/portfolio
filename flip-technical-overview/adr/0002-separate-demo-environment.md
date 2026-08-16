# ADR-0002: Separate Demo Environment

- **Status:** Accepted
- **Decision owner:** William Hargis

## Context

The production host `https://flip.engineering` serves real users and authorized production data. A technical demo must be reviewable without exposing real data, real accounts, production credentials, or production resources.

## Decision

The technical demo will run as a separate environment profile on `https://flip.tech-demo.dev`.

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

The demo will be seeded from versioned synthetic data only. It will use separate database/storage resources, separate capped credentials, and stable demo accounts. The environment must be reproducibly resettable from its seed set. The UI must show a persistent technical-demo banner. Desktop and web clients must select the demo profile explicitly.

## Consequences

- Reviewers can exercise retrieval, search, tools, citations, chat-to-forum synthesis, and relationship explanations without touching production.
- Production and demo cannot accidentally share data, credentials, or accounts.
- The demo is an explicit profile, not the default client target.
- A reset path and seed version must be maintained, or the demo drifts from reproducible behavior.
