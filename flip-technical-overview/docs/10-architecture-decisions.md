# Architecture decisions

This page records the main decisions that shape Flip's product and agent runtime. Each decision includes the operating reason, current cost, and conditions that would justify revision.

## 1. Keep the main product as a modular Phoenix application

**Decision:** Chat, forums, identity, authorization, search, synthesis, AI activity, media, jobs, and other product capabilities remain in one Phoenix and Ecto application with explicit context boundaries.

**Reason:** The main workflows cross product domains and require transactional coordination. An AI reply depends on identity, room visibility, chat state, model route, activity records, and final message persistence. Curation crosses chat, background work, forum publication, source relationships, and linkback. Splitting these paths across independently deployed services would add distributed transaction and authorization problems without a demonstrated scaling requirement.

**Cost:** Teams and modules must maintain clear application boundaries inside one repository. Expensive workloads require queue and process isolation. Large contexts can create coupling if domain ownership is not enforced.

**Revisit when:** A workload requires independent deployment or scaling, has a stable contract, and no longer needs cross-context transactions in the request path.

## 2. Use PostgreSQL as the durable authority

**Decision:** Durable product, agent, workflow, artifact, and client-synchronization state is committed to PostgreSQL. Realtime streams and provider conversations are not treated as the durable record.

**Reason:** Users and workers can disconnect or restart. Durable identity and lifecycle are required for deduplication, authorization, retry, recovery, continuation, and client convergence.

**Cost:** Workflows must define transaction boundaries and explicit state transitions. Streaming and optimistic clients need reconciliation against stored objects.

**Revisit when:** A specific state class has demonstrated requirements that PostgreSQL cannot meet and can be separated without weakening product consistency.

## 3. Represent AI as product identities and activities

**Decision:** AI participants have explicit product identities. Each execution has durable activity state related to the invoking actor, origin object, route, tools, sources, artifacts, and terminal outcome.

**Reason:** Attribution, access, diagnosis, and user-visible provenance require more than a provider response string. The system must distinguish the human request, AI identity, provider attempt, tool activity, and final product object.

**Cost:** The application maintains more lifecycle and provenance data than a simple chat-completions integration.

**Revisit when:** The identity or activity model no longer supports a required product surface. Revision should extend the model rather than move authority into provider metadata.

## 4. Separate direct AI authorship from conversation curation

**Decision:** Direct AI replies and curation of human messages use different workflows and provenance.

**Reason:** A direct reply creates new AI-authored content. Curation reorganizes selected human content and must retain source authorship, message references, destination planning, and correction state. Treating both as generic generation would make attribution and access unclear.

**Cost:** The product maintains separate data and workflow paths for related AI capabilities.

**Revisit when:** A new content mode has clearly defined authorship and provenance that cannot be represented by either existing path.

## 5. Keep capability selection and authorization on the server

**Decision:** The application builds a turn-specific capability catalog from actor scope, product surface, feature state, route support, and available services. The model can call only those capabilities.

**Reason:** A global tool list exposes irrelevant or unsafe operations and encourages the model to supply trusted scope. Product permissions and current service availability are application state.

**Cost:** Capability admission adds code and tests for each product surface and tool class.

**Revisit when:** A more general policy engine can express the same trusted inputs and refusal behavior without moving scope into model-generated arguments.

## 6. Split durable, asynchronous, ephemeral, and local client state

**Decision:** Durable product state, durable background state, ephemeral realtime state, and local client interaction state use separate ownership and delivery paths.

**Reason:** A message, an image-generation job, a typing indicator, and a draft have different replay, consistency, and recovery requirements. Using one transport or store for all four creates either unnecessary persistence or unrecoverable product behavior.

**Cost:** Clients and server code must reconcile several state channels and preserve canonical identities.

**Revisit when:** A state class changes its durability or replay requirements. The replacement still needs an explicit source of truth and ordering behavior.

## 7. Use provider-compatible inference behind product-owned routes

**Decision:** Flip defines a product request envelope and route requirements. Provider adapters translate that request to local or hosted services and normalize streams, tool calls, usage, terminal state, and failures.

**Reason:** Provider and model changes should not redefine product context, permissions, tool schemas, artifact identities, or persistence. Different workloads also need different route profiles.

**Cost:** Adapters must preserve meaningful provider differences and require route-specific evaluation.

**Revisit when:** A provider capability cannot be represented by the current request or event contract. Extend the contract with explicit semantics rather than bypassing the product runtime.

## 8. Store citations and artifacts as application objects

**Decision:** Sources, citations, files, charts, images, videos, polls, and other generated or retrieved artifacts receive durable application identities and lifecycle state.

**Reason:** The product must validate what a final reply refers to, display completion or failure, support later access checks, and continue workflows after the model turn ends.

**Cost:** Artifact and citation schemas, storage, cleanup, and access control require application code beyond text generation.

**Revisit when:** New artifact classes require different storage or lifecycle behavior. They should still expose stable application references to the agent runtime and clients.

## 9. Use durable jobs and continuation for long-running work

**Decision:** Operations that may outlive a request create durable jobs or artifacts. Terminal completion can start one deduplicated continuation under the original product scope.

**Reason:** Provider polling, callbacks, and media processing can complete after HTTP requests, workers, or application nodes restart. The original model session cannot be the only continuation mechanism.

**Cost:** The application must manage uniqueness, stale work, terminal events, partial failure, and continuation state.

**Revisit when:** A workload becomes reliably synchronous within the product deadline or requires a different multi-stage state machine.

## 10. Maintain a private implementation and a public technical portfolio

**Decision:** The implementation repository remains private. Public material contains architecture descriptions, diagrams, decisions, status, and synthetic technical scenarios without source links to the private repository.

**Reason:** Public technical review does not require access to product data, credentials, private deployment state, provider keys, prompt and persona state, or administrative controls. The portfolio should explain the engineering without creating an unavailable or misleading source path.

**Cost:** Public documentation must be maintained deliberately and cannot rely on readers following internal module links.

**Revisit when:** The product's source-availability policy changes. Any publication still requires a separate audit for data, secrets, licensing, operational authority, and environment-specific configuration.

## 11. Use a separate synthetic technical environment

**Decision:** Public scenarios use synthetic identities, communities, content, artifacts, and provider fixtures in a separate environment and namespace.

**Reason:** Architecture and failure behavior can be exercised without copying product data or granting product authority. A synthetic environment also supports deterministic cases for retries, access changes, provider failures, and client reconciliation.

**Cost:** Fixtures and scenario assertions require maintenance as product contracts change.

**Revisit when:** A different public evaluation method can provide equivalent contract coverage without exposing product state or coupling to production operations.

[Previous: Testing, evaluation, and operations](09-evaluation-testing-and-operations.md) · [Next: Status and limitations](11-roadmap-and-known-limitations.md)
