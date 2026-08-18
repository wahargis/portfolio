# ADR 0004 — Separate curation and AI authorship

- **Status:** Accepted
- **Decision scope:** Content integrity

## Context

Models are used both to organize human conversation into durable structure and to author new AI replies. Treating these as one workflow makes it unclear who said what.

## Decision

Model two paths:

1. **Curation:** preserve source-message and participant identity; use AI for topic structure, placement, and bounded bridge context.
2. **AI participation:** persist new content under an explicit AI identity with trigger, citation, artifact, and action provenance.

The UI and data model must keep source text, structural curation text, and AI-authored text distinguishable.

## Consequences

The product carries more content types and lifecycle rules, but it avoids silently rewriting community history or presenting AI composition as participant speech.

## Revisit when

No expected change. Implementations may evolve, but the authorship distinction is a product integrity invariant.
