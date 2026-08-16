# 00 — Executive Overview

Flip is a real-time chat and durable forum system with configurable AI participants, giving teams both the immediacy of live rooms and the permanence of forum threads.

<img src="../diagrams/system-context.svg" alt="Flip system context diagram" width="760" />

## Product thesis

Discussion and decision should not be separated. Chat rooms provide real-time delivery; forum threads provide durable, tagged records. When a room has synthesis enabled, an AI participant can curate a conversation into a forum outcome, and that outcome keeps a traceable link back to the chat messages that produced it.

## Architecture thesis

The product has a web client, a native desktop client, a server-side data and realtime layer, an asynchronous agent runtime for AI work, and external source retrieval. AI actions are bounded by round, time, and token limits and dispatched through isolated per-tool execution. Tools are context-gated, and generated claims are supported by quote-verified citations with a reader-facing source ledger.

## Live hosts

- Production: <https://flip.engineering>
- Technical demo: <https://flip.tech-demo.dev> (synthetic data, separate resources, capped credentials)

This repository is the public architecture path. The pages that follow describe what Flip does, how it is structured, and how its AI participants stay bounded and auditable — without publishing implementation source, prompts, credentials, or internal configuration.
