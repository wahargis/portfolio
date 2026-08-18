# ADR 0001 — Public architecture boundary

- **Status:** Accepted
- **Decision scope:** Portfolio publication

## Context

Flip is a commercial product with private implementation, deployment configuration, user data, provider credentials, prompts/personas, and abuse controls. A technical portfolio still needs enough detail to support serious architectural review.

A shallow marketing page is insufficient; a source mirror is inappropriate.

## Decision

Publish:

- product aims and actor model;
- domain, data, process, client, and deployment boundaries;
- sanitized lifecycles and failure semantics;
- architecture diagrams and stable decisions;
- implemented/evolving/experimental status;
- representative synthetic scenarios;
- honest limitations.

Do not publish:

- private source or source-derived secrets;
- production data or copied private messages;
- credentials, host paths, or security-sensitive thresholds;
- proprietary prompt/persona content;
- internal campaign logs, issue inventories, or implementation diaries;
- claims requiring evidence that is not included.

## Consequences

Reviewers can assess the product model and engineering decisions without receiving the commercial repository. Some source-level claims cannot be independently reproduced from this portfolio alone, so the documentation must avoid false precision and disclose that boundary.

## Revisit when

The implementation or selected components are intentionally open-sourced under a clear license and security review.
