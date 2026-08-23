# Status and limitations

This page separates current system behavior from engineering work that remains. It describes technical scope, not product usage or business performance.

## Current implementation

### Product and identity

The application includes accounts, community membership, chat rooms, messages and replies, reactions, polls, forums, search, media, notifications, settings, and administrative and moderation-related controls. AI identities participate through the same product model and are configured for selected surfaces and behavior.

### Agent runtime

The runtime admits turns from product events, resolves actor and origin scope, assembles bounded context, selects a route and capability catalog, executes model and tool rounds, validates terminal output, stores activity and references, and commits user-visible results.

Internal retrieval uses product scope. External research supports discovery and direct source reading. Document, PDF, structured data, chart, image, video, and other tools can create durable results and artifacts. Product actions pass through typed application services.

### Background work and continuation

Durable jobs and runs represent asynchronous work, retries, terminal state, and stale-work recovery. Long-running provider-backed artifacts can complete after the originating process and start one deduplicated continuation under stored product scope.

### Curation and provenance

Conversation curation has durable source selection, planning, destination resolution, forum mutation, source-message and participant relationships, linkback, feedback, and recuration state. Direct AI authorship and curation of human content remain distinct.

### Clients and operations

The system supports Phoenix web and realtime surfaces and a React-based desktop and mobile client. Durable synchronization, ephemeral channels, optimistic reconciliation, capability negotiation, activity records, structured failures, deterministic tests, and model-route evaluation are part of the architecture.

## Current limitations and engineering pressure

### Route semantics vary by provider

Provider adapters normalize common request and event behavior, but they cannot make all models and services equivalent. Tool-call formatting, stream interruption, context handling, media lifecycle, usage reporting, and structured-output reliability still require route-specific behavior and evaluation.

**Engineering work:** keep provider differences explicit in route metadata and tests; avoid fallback that changes privacy, tool, modality, or evidence requirements.

### Context selection is a continuing product problem

Bounded conversation and scoped retrieval prevent unbounded access, but relevance, compression, source ranking, and context-budget allocation can still improve. Large context windows increase available capacity but do not decide which product records should be included.

**Engineering work:** evaluate context selection by task and surface, improve traceable compaction, and retain direct links from included context to durable product objects.

### Tool coverage and effect policy are uneven

Read-only retrieval, analysis, product actions, and asynchronous generation have different risk and lifecycle profiles. Some capabilities have richer idempotency, approval, or repair behavior than others.

**Engineering work:** standardize capability metadata, effect identities, retry classes, operator repair paths, and denial behavior without erasing domain-specific rules.

### Asynchronous workflows create partial outcomes

Media, documents, and curation can complete some stages and fail later stages. A single success or failure flag is insufficient.

**Engineering work:** continue moving complex operations toward explicit stage state, stage-specific retry, durable compensation or repair, and user-visible partial results.

### Client convergence needs continuing end-to-end testing

Server-rendered web, realtime channels, APIs, and durable native synchronization can deliver related state in different orders. Permission changes and optimistic objects add additional cases.

**Engineering work:** expand scenario coverage for reordered delivery, reconnect, revoked access, duplicate commands, stale clients, background completion, and cross-client artifact updates.

### Evidence quality is route- and task-dependent

The system can store sources and citations, but source discovery, direct-read quality, document parsing, citation placement, and model use of evidence vary by workload.

**Engineering work:** maintain retrieval and citation evaluation sets, validate references against stored evidence, expose incomplete evidence state, and distinguish source quality from answer fluency.

### Evaluation does not reduce to one score

Conversation quality, research accuracy, tool behavior, product effects, latency, cost, privacy, and recovery require different measurements. A route that performs well on one surface may be unsuitable for another.

**Engineering work:** maintain route- and surface-specific evaluation, deterministic product invariants, failure injection, and operational evidence instead of one global model ranking.

### The private implementation limits direct source review

The public portfolio cannot rely on readers opening private modules or product environments.

**Engineering work:** keep public diagrams, decisions, synthetic scenarios, status, and architecture descriptions detailed enough to support technical review without private source links or internal data.

## Boundaries not claimed

- The documentation does not claim public repository access for the Flip implementation.
- It does not claim a specific community scale, latency, reliability percentage, or model-quality score without published measurement.
- It does not claim that every tool has identical approval, retry, or compensation behavior.
- It does not claim that all provider routes support the same context, tools, modalities, or interruption semantics.
- It does not claim that curation automatically produces a correct publication without review and correction paths.
- It does not claim that the synthetic environment is a production mirror.

## Public technical material

The public Flip material includes this technical series, architecture decision records, diagrams, and the synthetic environment specification. Product data, user content, private source, credentials, provider keys, host identifiers, deployment topology, prompt and persona state, and administrative authority remain outside the portfolio.

[Previous: Architecture decisions](10-architecture-decisions.md) · [Back to the Flip case study](../README.md)
