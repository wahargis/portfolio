# ADR 0001 — Public architecture boundary

- **Status:** Accepted
- **Decision scope:** Portfolio publication

## Context

Flip's implementation repository is private. The public portfolio still needs enough technical evidence to explain the product, agent runtime, data model, authorization, client architecture, workflows, and failure handling without publishing private code or replacing the implementation repository with a partial copy.

A shallow product description would conceal the engineering. A source mirror would expose unnecessary implementation and operational detail, create maintenance drift, and weaken the private-repository boundary.

## Decision

Publish selected public architecture material:

- product and domain behavior;
- system, data, process, client, and deployment boundaries;
- sanitized lifecycle and failure semantics;
- rendered architecture diagrams and stable decisions;
- representative product and synthetic-environment images;
- private source paths that identify where load-bearing behavior is implemented;
- implemented scope and current limitations.

Do not publish or duplicate:

- private source code;
- production data or copied private messages;
- credentials, host paths, tokens, or security-sensitive thresholds;
- production deployment and administrative state;
- proprietary prompt or persona content not required to understand the architecture;
- internal campaign logs or issue chronology as the project narrative;
- claims that cannot be supported by public architecture or selected implementation evidence.

## Consequences

The public repository can establish the system's architecture and engineering depth while preserving source and operational confidentiality. Source-path references must remain synchronized with the private implementation, and public diagrams must avoid becoming generic substitutes for the actual behavior they describe.

## Revisit when

Revisit if the implementation repository becomes public, the confidentiality boundary changes, or a distributable reference implementation replaces the current portfolio evidence model.
