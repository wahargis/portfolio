# 04 — Retrieval, Search, and Tools

Flip gives AI participants governed access to external sources through search, retrieval, and read tools.

Source discovery is handled by `Flip.Search.SourceDiscovery`, which ranks source types and profiles, groups domains, and supports preferred-domain discovery. `Flip.Search.BraveClient` wraps the Brave Search API with a circuit breaker. `Flip.Retrieval.Cascade` provides staged retrieval helpers, so searches can degrade or escalate through retrieval stages rather than issuing unbounded calls.

The tool layer is context-gated. `Flip.Synthesis.Tools.available_tools/1` and `apply_capability_context/1` define capability contexts including `:synthesis`, `:personal_ai_reply`, `:forum_ai_reply`, `:forum_enrichment`, and `:game_turn`. `execute/2` accepts an `authz_scope` and fails closed for search tools when the scope does not authorize them. `ToolLoop` enforces the advertised-tool allowlist before dispatch.

Outbound web access is filtered by `Flip.Synthesis.UrlGuard`, which maintains an SSRF denylist. Read tools include `read_webpage`, `research_session`, and `wayback`; `webpage_reader` and `Flip.Archive.Wayback` support page and archive reads.

The design pattern is consistent: a model is told about a narrow set of tools, each tool call is checked against the advertised set, the execution scope must authorize the tool, the URL must pass the guard, and the actual dispatch is isolated and time-bound. External searches are protected by a circuit breaker. This is defense in depth around retrieval rather than a single prompt instruction.

<img src="../diagrams/retrieval-source-citation-flow.svg" alt="Retrieval, search, and citation flow" width="760" />
