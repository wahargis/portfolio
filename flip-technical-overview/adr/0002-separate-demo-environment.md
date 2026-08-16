# Separate Demo Environment

The production host `https://flip.engineering` serves real users and authorized production data. A technical demo must be reviewable without exposing real data, real accounts, production credentials, or production resources.

## Principle

The technical demo runs as a separate environment on `https://flip.tech-demo.dev`.

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

## Expected effect

Reviewers can exercise retrieval, search, tools, citations, chat-to-forum synthesis, and relationship explanations without touching production. Production and demo cannot accidentally share data, credentials, or accounts, and the demo remains an explicit profile rather than the default client target.
