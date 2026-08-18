# 10 — Architecture Decisions

This page summarizes the product’s consequential choices. Individual public ADRs provide the stable rationale for the most important decisions.

## Decision table

| Decision | Rationale | Cost / tradeoff |
|---|---|---|
| **Combine chat and forum in one product** | Live exchange and durable knowledge are one lifecycle; shared identity/provenance enables direct transition. | Larger product surface and more cross-domain invariants. |
| **Use a modular Phoenix monolith** | Shared transactions, authorization, PubSub, jobs, and migrations are more valuable than service isolation at current scale. | Requires disciplined contexts and can produce large modules if boundaries erode. |
| **Keep PostgreSQL canonical** | Strong relational provenance, full-text search, Oban state, and synchronization from one durable store. | Database scaling and migration discipline become central. |
| **Use Oban for asynchronous workflows** | Durable retries, uniqueness, scheduling, and inspection without an external message broker. | Background throughput depends on database health and careful job contracts. |
| **Separate curation from AI participation** | Human authorship preservation and AI-authored answers require different integrity rules. | Two lifecycle models and more explicit UI/content types. |
| **Make AI identity explicit** | Users should know when content is AI-authored and which configured participant produced it. | Persona/model/configuration state must remain coherent across surfaces. |
| **Compute tool catalogs server-side** | Prompt instructions are not authorization; the code must decide which capabilities exist. | Catalog logic and parity tests become substantial. |
| **Propagate trusted actor/community scope into tool tasks** | Internal retrieval must preserve product permissions even under asynchronous dispatch. | Scope plumbing must be explicit through every execution path. |
| **Fail closed for internal retrieval** | Missing scope should never become global search. | Some recoverable misconfiguration produces empty results instead of best-effort behavior. |
| **Persist citations and artifacts as objects** | Evidence and generated outputs need durable identity, validation, lifecycle, and UI. | Additional schemas and cleanup/retention obligations. |
| **Use different realtime paths for durable and ephemeral state** | Electric suits recoverable records; channels suit presence/typing and transient progress. | Clients must coordinate several transports and reconnection semantics. |
| **Keep server-authoritative mutations** | Product permissions and invariants must not be delegated to local client state. | Offline mutation requires queues and reconciliation rather than direct local writes. |
| **Use provider-compatible inference boundaries** | Models and providers should be replaceable without changing product semantics. | Adapters must normalize incompatible tool/stream/finish behavior. |
| **Reserve a terminal composition stage** | A tool-using model can otherwise finish with internal intent or protocol markup instead of a user reply. | Extra latency/token cost and provider-specific compatibility work. |
| **Separate product and synthetic technical environments** | Architecture can be reviewed without private data or credentials. | Additional deployment/seed maintenance. |
| **Publish architecture, not private source** | Reviewers can assess design without exposing commercial implementation and deployment internals. | Some claims cannot be independently code-reviewed from this repository alone. |

## Decision criteria

The decisions optimize for:

1. **authorship integrity;**
2. **authorization correctness;**
3. **durable provenance;**
4. **recoverability under asynchronous failure;**
5. **model/provider replaceability;**
6. **one canonical product state;**
7. **reviewability without private-data exposure.**

Raw feature count is not the optimization target.

## Rejected simplifications

### “Just summarize chat into a post”

Rejected because it loses participant wording, message identity, reply structure, and reader navigation to the source.

### “Give the model a database tool”

Rejected because model-selected SQL or generic CRUD cannot express product authorization, idempotency, audit, and effect-specific workflows safely.

### “Use one websocket for everything”

Rejected because durable synchronized records and ephemeral presence have different replay, offline, and consistency requirements.

### “Let the client write the local database and sync later”

Rejected as a universal model because membership, moderation, and cross-user conflict semantics remain server-authoritative. Selected offline queues can be added per command.

### “One best model for all AI features”

Rejected because curation, low-latency reply, tool use, long documents, structured planning, and multimodal tasks have different route requirements.

### “Move every domain into a service”

Rejected until scaling or team ownership justifies distributed consistency and duplicated operational boundaries.

## Revisit triggers

An architectural decision should be revisited when evidence changes:

- product scale makes one PostgreSQL/Oban topology insufficient;
- client offline mutation requirements become explicit and conflict semantics are defined;
- a domain requires independent release or regulatory isolation;
- a provider-neutral standard eliminates meaningful adapter complexity;
- curation quality requires a separate specialized model service;
- artifact/media workloads need a dedicated execution plane;
- public source release becomes compatible with commercial and security boundaries.

Revisit does not mean erase the original rationale. The ADR record should preserve why the prior choice was correct under prior constraints.
