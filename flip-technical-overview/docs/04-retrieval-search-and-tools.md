# 04 — Retrieval, Search, and Tools

Flip gives AI participants governed access to external sources through source discovery, search, retrieval, and read tools.

<img src="../diagrams/retrieval-source-citation-flow.svg" alt="Retrieval, search, and citation flow" width="760" />

## Source discovery and retrieval

A source-discovery layer ranks source types and domains, groups related sources, and supports preferred-domain discovery. Retrieval is staged: searches can degrade or escalate through retrieval stages instead of issuing unbounded calls. External search access is protected by a circuit breaker so repeated provider failures do not accumulate retries.

## Context-gated tools

The tool layer is context-gated. Each capability context defines which tools are available for that kind of work — for example, synthesis, personal AI replies, forum AI replies, forum enrichment, or game turns. Tool execution checks the requested authorization scope and fails closed for search tools when the scope does not authorize them. The runtime enforces an advertised-tool allowlist before dispatch.

## Safe outbound access

Outbound web access is filtered by a URL guard that blocks unsafe destinations. Read tools cover live pages and archived captures, with page readers and archive retrieval behind the same guard. The design pattern is consistent: the model sees a narrow set of tools, each call is checked against the advertised set, the scope must authorize it, the URL must pass the guard, and dispatch is isolated and time-bound.
