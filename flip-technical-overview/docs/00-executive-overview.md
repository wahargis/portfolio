# 00 — Executive Overview

Flip is a hybrid real-time chat and durable forum system with configurable AI participants. Teams get the immediacy of live rooms and the permanence of forum threads without splitting their discussion across tools.

The product thesis is that discussion and decision should not be separated. Chat rooms provide real-time delivery through Phoenix channels. Forum threads provide durable, tagged records backed by `Flip.Chat` and `Flip.Forum`. When a room has synthesis enabled, an AI participant can curate the conversation into a forum outcome, and that outcome keeps a traceable link back to the chat messages that produced it.

The architecture thesis is a Phoenix LiveView and Phoenix Channel web surface, an Oban-backed agent runtime for AI work, and ElectricSQL synchronization to a Tauri desktop client. AI actions are bounded by `Flip.Synthesis.ToolLoop` (round cap, wall-clock deadline, input-token envelope) and dispatched through `Flip.Synthesis.IsolatedDispatch`. Tools are context-gated, and generated claims are supported by AI-minted, quote-verified citations with a reader-facing source ledger.

Two live hosts are available:

- Production: <https://flip.engineering>
- Technical demo: <https://flip.tech-demo.dev>

This repository is the public architecture path. The pages that follow describe what Flip does, how it is structured, and how its AI participants stay bounded and auditable — without publishing implementation source, prompts, credentials, or internal configuration.
