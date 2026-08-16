# 10 — Architecture Decisions

This page summarizes the public design principles that shape the Flip platform. The short-form principle records live in [adr/](../adr/README.md).

## Public architecture boundary

Flip’s implementation is not published in this repository. This overview describes structure, capabilities, and operational behavior without publishing source code, prompts, credentials, or internal URLs.

## Separate demo environment

The technical demo runs on a separate host with synthetic seed data, separate resources, capped credentials, stable demo accounts, reproducible reset, an explicit demo client profile, and a persistent demo banner.

## Bounded agent runtime

AI work uses round caps, deadlines, token envelopes, forced finish mode, and isolated per-tool dispatch instead of unbounded loops. Finish-mode timeouts are non-retryable to avoid replaying side-effectful tools.

## Context-gated tools

Tools are advertised per capability context, enforced by an advertised-tool allowlist at dispatch, authorized by scope, filtered by a URL guard, and isolated in execution. Search tools fail closed.

## Provenance by construction

Forum and chat records carry source attribution; synthesis threads must reference their source discussion; citations are quote-verified and exposed through a source ledger.

## Encrypted provider keys and deterministic routing

Model provider credentials are encrypted, the active provider is selected through settings, and media generation uses a deterministic router.

## Save-first recovery

Generated media is staged through recovery paths before final placement, with audit records capturing operational events.
