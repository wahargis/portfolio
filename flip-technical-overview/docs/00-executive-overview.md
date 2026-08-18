# 00 — Executive Overview

## System in one paragraph

Flip is a real-time community product that combines chat, threaded forums, background conversation curation, and explicit AI participants. Humans can converse at chat speed; durable knowledge can be organized into forum structure without erasing the source conversation; and AI can research, answer, create artifacts, and take permitted actions through a bounded server-side runtime. PostgreSQL is the transactional source of truth, Oban runs asynchronous workflows, Phoenix provides the web and realtime server, Electric synchronizes durable client state, and provider-compatible inference keeps product semantics independent of one model vendor.

## The engineering problem

A conventional chatbot integration usually assumes:

```text
user prompt -> model response -> render text
```

That is insufficient inside a multi-user product. A production participant must be constrained by:

- who invoked it;
- what community and room the request belongs to;
- what content that actor may read;
- which tools are available in that surface;
- what effects require product authorization;
- how current information is sourced and cited;
- how duplicate jobs are prevented;
- what happens when a tool, provider, or long-running artifact fails;
- how the result becomes durable product state;
- how other clients learn about the state change.

Flip treats those as first-class product architecture.

## Four product loops

### 1. Human conversation

A user writes in chat or forum. The server applies membership, authorization, content, and rate constraints; commits the durable record; and distributes the update through the appropriate realtime path.

### 2. Conversation curation

Eligible chat material is selected, grouped by topic, and mapped into forum structure. User-authored text remains linked to source messages. Structural bridge text, placement, deduplication, and linkback are generated under a separate synthesis workflow.

### 3. AI participation

A mention, reply, scheduled action, or product-specific event enqueues an AI turn. The runtime assembles bounded context, computes the tool catalog, routes a model request, executes permitted tools, records citations/artifacts, obtains a terminal draft, and persists an attributed AI response.

### 4. Artifact continuation

Some tools produce asynchronous work—such as media generation or multi-clip workflows. The immediate turn records a durable request and later terminal events can re-enter a bounded AI continuation so the participant can interpret the result in product context.

These loops share identity, permissions, content, and provenance but have different authorship and lifecycle rules.

## Architectural center of gravity

Flip is a modular monolith rather than a microservice mesh. Product domains remain explicit, but they share:

- one PostgreSQL transaction boundary;
- one authorization model;
- one PubSub system;
- one Oban job system;
- direct foreign-key relationships between source conversation and durable knowledge;
- one release and migration path.

This is deliberate. Splitting chat, forum, synthesis, and AI state into independent services would make provenance and cross-domain consistency harder without solving a demonstrated scaling problem.

## Data and state classes

| State class | Examples | Primary mechanism |
|---|---|---|
| **Transactional durable state** | users, memberships, messages, threads, replies, votes, citations, artifacts, job records | PostgreSQL/Ecto |
| **Asynchronous workflow state** | synthesis run, AI reply, recuration, artifact generation, recovery | Oban + PostgreSQL |
| **Durable client synchronization** | messages, threads, settings, read models | Electric shapes / HTTP mutations |
| **Ephemeral realtime state** | typing, presence, transient progress | Phoenix channels/PubSub |
| **External evidence** | web pages, documents, data series, media provider output | tool-specific records and provenance |
| **Model conversation envelope** | selected context, tool calls/results, terminal draft | bounded runtime state, then durable result |

Confusing these state classes leads to common design defects: trying to persist typing through the main data model, treating a background job as a chat message, trusting the client as mutation authority, or retaining model context as if it were product provenance.

## AI runtime boundary

The runtime is intentionally split into code-owned stages:

1. detect and normalize the trigger;
2. derive actor/community authorization scope;
3. reserve uniqueness/concurrency capacity;
4. assemble product and conversation context;
5. compute the capability catalog;
6. call the selected provider;
7. parse and validate tool calls;
8. execute tools in isolated tasks;
9. append structured results and citations;
10. repeat under explicit control bounds;
11. force or repair terminal composition when needed;
12. validate user-visible output;
13. persist the attributed reply and artifacts;
14. publish product updates and telemetry;
15. recover or disclose terminal failure honestly.

The model can choose among admitted tools. It cannot expand its own catalog or bypass product authorization.

## Trust and provenance

Flip has several provenance systems because “source” means different things in different workflows:

- **conversation provenance:** which chat messages and participants produced a forum artifact;
- **citation provenance:** which quote or document passage supports an AI-authored claim;
- **artifact provenance:** which request, tool call, provider result, and continuation produced an artifact;
- **action provenance:** which actor, AI participant, room, and product event caused an effect;
- **operational provenance:** which job attempt, model route, and recovery path produced the durable outcome.

The public architecture emphasizes the first four. Sensitive thresholds and private telemetry details remain outside the portfolio.

## Product differentiators

1. **Chat and forum are one data product, not linked external services.**
2. **Curation and AI authorship are different workflows with different guarantees.**
3. **AI tools include product-native actions, internal retrieval, external research, structured data, and media—not only web search.**
4. **Authorization scope follows tool execution into child tasks and fails closed when absent.**
5. **Artifacts and citations are durable objects, not decorations embedded in a model response.**
6. **Provider compatibility allows local or hosted inference without moving product authority into the inference layer.**
7. **The same product model supports web, desktop, and mobile projections.**

## Review path

For a technically representative review:

1. read [System Architecture](02-system-architecture.md);
2. trace [Agent Runtime](03-agent-runtime.md);
3. compare the two paths in [Synthesis and Provenance](05-synthesis-and-provenance.md);
4. inspect the tool/authorization model in [Retrieval, Search, and Tools](04-retrieval-search-and-tools.md);
5. review [Status and Limitations](11-roadmap-and-known-limitations.md).

Deployment separation and quality mechanics are supporting material, not the primary product story.
