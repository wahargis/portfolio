# 08 — Deployment Topology

## Purpose of separate environments

Flip’s product architecture can be reviewed without access to private data or production credentials. A separate synthetic technical environment exists to exercise representative workflows with stable, non-sensitive records.

<img src="../diagrams/deployment-topology.svg" alt="Product and synthetic deployment topology" width="900" />

The environment split is supporting architecture, not the main portfolio narrative.

## Environment roles

| Environment | Purpose | Data |
|---|---|---|
| **Product** | Real users, communities, content, configured AI participants, and product operations. | Private product data under deployment policy. |
| **Synthetic technical environment** | Reproducible architecture scenarios, reviewer-safe examples, and selected integration checks. | Versioned synthetic users, rooms, threads, sources, and artifacts. |
| **Automated test/CI** | Deterministic unit, integration, migration, client, and security checks. | Ephemeral fixtures and generated test data. |
| **Local development** | Feature development and debugging. | Developer-owned local fixtures; no implicit production copy. |

## Shared architecture, separate authority

The product and synthetic environments can share:

- source version;
- migration history;
- domain model;
- job definitions;
- provider adapter contracts;
- public scenario definitions;
- observability schema.

They must not share:

- database;
- credentials;
- session/token secrets;
- object/media storage;
- provider keys unless separately provisioned;
- encryption/signing secrets;
- user uploads;
- queues or PubSub namespaces;
- administrative state.

The environment identifier is infrastructure configuration, not a field the model can change.

## Synthetic data design

Useful synthetic data represents relationships, not merely row counts:

- multiple communities and rooms;
- members with different visibility;
- a live chat exchange with reply structure;
- a forum topic derived from chat;
- direct and AI-authored replies;
- citations from safe public sources;
- pending/completed/failed artifacts;
- feature/permission differences;
- a correction or recuration example.

This allows reviewers to inspect authorization, provenance, lifecycle, and synchronization instead of only a polished happy-path screenshot.

## Provider and inference separation

Each environment has its own route policy and credentials. The synthetic environment may use:

- a lower-cost hosted route;
- a local HomeCloud endpoint;
- deterministic fakes for selected tests;
- restricted media providers;
- reduced concurrency.

Provider differences must not change the schema or authorship/provenance contract.

## Reproducibility

A synthetic scenario should be reconstructible through:

- versioned seed definitions;
- deterministic or bounded generated identities;
- migration-driven schema;
- explicit fixture version;
- repeatable cleanup of scenario-specific state;
- no dependency on production snapshots.

Operational reset commands and secrets are not public documentation. The public contract is that scenarios use synthetic state and can be reconstructed without production access.

## Security boundary

The synthetic environment is still an internet-facing application and should retain:

- authentication and authorization;
- rate and abuse controls;
- CSRF/session protections;
- URL/fetch restrictions;
- tool admission;
- provider-key secrecy;
- output sanitization;
- dependency and image scanning;
- logging redaction.

“Demo” must not mean bypassing the architecture being demonstrated.

## Review scenarios

The environment should make these architectural questions visible:

1. Does internal retrieval respect actor/community scope?
2. Does an external-research answer expose citations and source records?
3. Does curation preserve source-message authorship?
4. Can a reader navigate between chat and forum provenance?
5. Does an artifact have pending/completed/failed lifecycle?
6. Do realtime clients converge after mutation?
7. Does provider/tool failure produce honest product state?

The scenarios are documented in [demo/README.md](../demo/README.md).

## Endpoint independence

The case study and diagrams remain the authoritative public explanation. Endpoint availability, temporary maintenance, or route configuration should not determine whether the architecture can be reviewed.
