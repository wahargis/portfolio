# 10 — Architecture Decisions

This page records the architecture decisions that shape Flip. The formal records for the two public-boundary decisions are in `adr/`.

**Public architecture boundary.** Flip's implementation is not published in this repository. This overview describes structure, capabilities, and operational behavior without publishing source code, prompts, credentials, or internal URLs. ADR-0001 records this boundary.

**Separate demo environment.** The technical demo runs on a separate host with synthetic seed data, separate resources, capped credentials, stable demo accounts, reproducible reset, an explicit demo client profile, and a persistent demo banner. ADR-0002 records the separation table.

**Bounded agent runtime.** AI work uses `ToolLoop` round caps, deadlines, token envelopes, forced finish mode, and isolated per-tool dispatch instead of unbounded loops. Finish-mode timeouts are non-retryable to avoid replaying side-effectful tools.

**Context-gated tools.** Tools are advertised per capability context, enforced by an advertised-tool allowlist at dispatch, authorized by `authz_scope`, filtered by `UrlGuard`, and isolated by `IsolatedDispatch`. Search tools fail closed.

**Provenance by construction.** Forum and chat records carry source fields; synthesis threads must reference their source channel and message; citations are quote-verified and exposed through a source ledger.

**Encrypted provider keys and routing.** LLM provider keys are encrypted, the provider is selected through app settings, and a deterministic video router maps requests to providers.

**Save-first recovery.** Generated media is staged through recovery spools before final placement, with audit logs recording operational events.

Decisions are recorded as facts about the system when they are implemented and observable in behavior or configuration.
