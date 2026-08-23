# Product and synthetic environment boundary

Flip's public technical environment is separated from the product deployment. It exercises application contracts with synthetic data and fixtures without copying product accounts, messages, credentials, provider configuration, or administrative authority.

![Product and synthetic environment topology](../diagrams/deployment-topology.svg)

## Shared contracts

The synthetic environment can use the same kinds of schemas and state transitions for:

- account, membership, room, message, forum, and artifact records;
- AI identity and activity state;
- actor- and origin-aware authorization;
- context, retrieval, tool, effect, and terminal-response contracts;
- curation runs and source provenance;
- Oban jobs, retry, stale-work recovery, and continuation;
- API, realtime, synchronization, and client reconciliation;
- provider fixtures and route evaluation inputs.

The purpose is to exercise behavior, not to reproduce product data or capacity.

## Required separation

| Resource | Product environment | Synthetic environment |
|---|---|---|
| **Database** | Product accounts, communities, content, activities, artifacts, and operations. | Synthetic-only identities, content, activities, and scenario state. |
| **Queues and workers** | Product jobs and provider operations. | Scenario-specific jobs and fixtures with separate queue namespace. |
| **Object storage** | Product uploads and generated artifacts. | Synthetic files and generated test artifacts in separate storage roots. |
| **Authentication** | Product sessions, keys, and identity providers. | Synthetic accounts and independently scoped credentials. |
| **Providers** | Product route policy and keys. | Fixtures, stubs, or independently scoped test keys. |
| **Callbacks and webhooks** | Product hostnames and route secrets. | Separate hostnames, secrets, and callback validation. |
| **Push, email, and deep links** | Product delivery channels. | Disabled, stubbed, or separately scoped technical channels. |
| **Telemetry** | Product operational and user activity. | Synthetic scenario traces and test output. |
| **Administrative access** | Product operator authority. | Synthetic-only administration with no product reach. |

The synthetic application should fail when a product-only resource is referenced. It should not silently proxy the request to the product deployment.

## Provider fixtures

Deterministic fixtures can model:

- normal streaming replies;
- valid and invalid tool calls;
- repeated or malformed deltas;
- refusal and structured-output failure;
- timeout, disconnect, quota, and retryable unavailability;
- asynchronous image or video progress and completion;
- duplicate callbacks and late terminal events;
- usage present, missing, or inconsistent;
- a fallback route with different capability constraints.

Fixtures make failure and recovery paths repeatable. Live-provider evaluation can run separately with non-product data and independently scoped credentials.

## Synthetic product scenarios

The environment includes scenarios for:

- direct AI participation in an allowed room;
- denied access and revoked membership;
- internal retrieval with a nearby restricted record;
- web discovery, direct source reading, and citation validation;
- typed product effects and duplicate command handling;
- long-running artifact completion after application restart;
- curation with destination validation, provenance, and partial failure;
- native-client reconnect and reordered command, sync, and realtime delivery.

Each scenario should assert durable application state, not only generated text.

## Exported evidence

Technical evidence can include:

- architecture diagrams and state-transition descriptions;
- synthetic object and event timelines;
- test summaries and failure-injection results;
- provider-fixture traces with sensitive fields removed;
- client-convergence state before and after reconnect;
- curation source and destination relationships using synthetic content;
- artifact lifecycle and continuation records.

Exports exclude private prompts, credentials, raw provider keys, internal hostnames, copied product identifiers, and any content derived from product users.

## Network and configuration controls

Where practical, the synthetic environment should use:

- separate environment variables and secret stores;
- separate database, object storage, and queue endpoints;
- deny lists or network policy for product hostnames and callbacks;
- synthetic-only domain names and deep-link schemes;
- independent provider keys with restricted quotas;
- startup validation that rejects product resource identifiers;
- output scanning for secret-shaped values and internal hostnames.

Configuration separation is enforced by the application and deployment, not only by naming conventions.

## Limits of the synthetic environment

The environment does not prove product scale, production availability, or the quality of live product data. Provider fixtures do not replace live route evaluation. Synthetic authorization cases do not replace security review of the product deployment.

It provides controlled evidence for system contracts and failure behavior while maintaining the private implementation and product boundary.

[Previous: Model routing and inference](07-model-routing-and-inference.md) · [Next: Testing, evaluation, and operations](09-evaluation-testing-and-operations.md)
