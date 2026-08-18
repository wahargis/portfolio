# ADR 0001 — Curated portfolio boundary

- **Status:** Accepted
- **Decision scope:** Portfolio publication

## Context

Flip has a public source repository, but a technical portfolio still needs a curated explanation of the product rather than a second, partial copy of the source tree. The source also contains implementation chronology, deployment-specific configuration, tests, provider integrations, and operational detail that do not all belong in the reviewer path.

A shallow marketing page is insufficient; duplicating the repository is unnecessary and creates drift.

## Decision

The portfolio publishes:

- product aims and actor model;
- domain, data, process, client, and deployment boundaries;
- sanitized lifecycles and failure semantics;
- architecture diagrams and stable decisions;
- implemented/evolving/experimental status;
- representative synthetic scenarios;
- honest limitations;
- a link to the canonical public source repository.

The portfolio does not publish or duplicate:

- production data or copied private messages;
- credentials, host paths, tokens, or security-sensitive thresholds;
- deployment-specific secrets and private operational state;
- prompt/persona content that is not required to understand architecture;
- internal campaign logs or issue chronology as the main narrative;
- claims requiring evidence that is not included.

## Consequences

Reviewers can move from a stable architecture narrative to the canonical source when they need implementation detail. The portfolio remains readable and avoids drifting into a stale mirror.

## Revisit when

The public source structure or portfolio audience changes enough that a different review path is more useful.
