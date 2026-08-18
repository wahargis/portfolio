# 10 — Architecture Decisions

## The decisions are organized around product integrity

Flip’s architecture is not a collection of fashionable technologies. Its major choices follow from three product requirements:

1. live conversation and durable knowledge must remain connected;
2. AI participation must not erase authorship or bypass permissions;
3. asynchronous and model-driven work must still converge on one inspectable product state.

The public ADRs preserve the stable rationale. This page shows how the choices fit together.

## 1. Chat and forum are one product lifecycle

**Decision:** Keep live rooms and durable threaded discussion in one product and one identity/provenance model.

**Why:** Chat captures the social and causal path of a discussion; forums make the result searchable and extendable. Separating them into unrelated products would make the transition manual and provenance fragile.

**Cost:** Flip must maintain two interaction models and the cross-domain rules that connect them.

**Rejected alternative:** “Summarize the chat into a post.” A summary alone loses participant wording, reply structure, source navigation, and correction history.

## 2. Use a modular Phoenix monolith with PostgreSQL authority

**Decision:** Keep product domains in one Phoenix application, one migration path, one authorization model, and one canonical PostgreSQL database. Use Oban for durable asynchronous work.

**Why:** Message-to-forum provenance, AI replies, citations, artifacts, linkback, notifications, and client synchronization all benefit from shared identities and transactions. An external message broker or early service split would add network failure and eventual consistency before a demonstrated need.

**Cost:** The application can accumulate large contexts and workers. Boundary tests and refactoring discipline are mandatory.

**Revisit when:** A domain demonstrates an independent scaling, regulatory, release, or team-ownership requirement that outweighs distributed consistency cost.

## 3. Separate curation from AI authorship

**Decision:** Model conversation curation and direct AI participation as different workflows and content semantics.

**Why:** Curation should preserve human words and source identity while changing structure. An AI reply is new composition and must be attributed to the AI participant. One generic “synthesis” path makes authorship ambiguous.

**Cost:** Two lifecycle models, explicit content types, and more provenance-aware UI.

**Rejected alternative:** Rely on a prompt saying “do not rewrite users.” Product integrity must survive model error and therefore requires durable source relationships.

## 4. Keep the capability plane server-authoritative

**Decision:** Compute the available tool catalog from surface, feature state, actor/community authorization, object context, and provider availability. Pass trusted scope into child tool tasks and fail closed for protected retrieval.

**Why:** Prompt instructions are not permission enforcement. Internal search, product actions, and provider-backed effects need the same authorization and audit guarantees as ordinary user commands.

**Cost:** Catalog coherence, effect classification, scope propagation, and parity tests become substantial engineering work.

**Rejected alternative:** Give the model a generic database or CRUD tool. That would bypass domain validation, idempotency, and product-specific effect lifecycles.

## 5. Persist evidence and artifacts outside model prose

**Decision:** Citations, source records, charts, generated media, polls, and asynchronous requests receive durable product identities and lifecycle state.

**Why:** A token or URL embedded only in a model response cannot be validated, deduplicated, permissioned, retried, continued, or rendered reliably. Durable objects make evidence and effects inspectable after the model context is gone.

**Cost:** More schemas, storage, cleanup, retention, and UI state.

**Rejected alternative:** Treat generated output as opaque markdown attachments. That hides pending/failure state and loses causal inputs.

## 6. Split durable synchronization from ephemeral realtime

**Decision:** Use server-authoritative commands for mutations, Electric for recoverable durable projections, and Phoenix channels/PubSub for presence, typing, and transient progress.

**Why:** Durable messages and artifacts need replay and reconnect; ephemeral interaction does not. One websocket stream cannot provide both semantics cleanly.

**Cost:** Clients coordinate several transports and must test ordering and reconnection explicitly.

**Rejected alternative:** Let the client own a local product database and sync later by default. Membership, moderation, and cross-user conflicts remain server-authoritative; offline mutation must be specified per command.

## 7. Keep models and providers replaceable

**Decision:** Place model routing and provider adapters behind a product-owned request, tool, evidence, and terminal-composition contract.

**Why:** Chat, curation, research, documents, and media have different execution needs, and provider APIs change. Product identity and permissions should not change when a route changes or local inference replaces a hosted endpoint.

**Cost:** Adapters must normalize incompatible streaming, tool-call, finish, forced-tool, and error behavior. Routes require surface-specific evaluation.

**Rejected alternative:** Choose one “best model” for every feature. A route strong at long research may be unsuitable for low-latency chat or structured curation.

## 8. Reserve an explicit terminal composition step

**Decision:** Distinguish working/tool rounds from the user-visible reply and validate the terminal draft before persistence.

**Why:** Tool-using models can end with internal intent, hidden protocol, or an incomplete draft. The product needs a bounded point where gathered evidence becomes the actual attributed response.

**Cost:** Additional latency and provider-specific compatibility work.

## 9. Separate public technical review from product authority

**Decision:** Maintain a synthetic environment and publish architecture, scenarios, decisions, and limitations without copying production data, credentials, or private source.

**Why:** Reviewers need relationally realistic evidence, not production access. A public architecture portfolio should be technically meaningful without becoming a source or security dump.

**Cost:** Synthetic fixtures and a separate deployment require maintenance, and not every source-level claim is independently reproducible here.

## Decision criteria

Across these choices, Flip optimizes for:

- visible and correct authorship;
- server-enforced authorization;
- durable provenance and correction;
- coherent failure and recovery;
- one canonical product state;
- model/provider replaceability;
- client convergence;
- reviewability without private-data exposure.

Feature count is not the optimization target.

## Revisit discipline

Architecture should change when evidence changes: measured database or queue limits, explicit offline conflict requirements, independent regulatory boundaries, route evaluation, artifact workload isolation, or a deliberate public-source strategy.

A change should preserve the original decision record rather than rewriting history. The ADRs explain why the current choice was correct under the constraints that produced it.