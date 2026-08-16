# ADR-0001: Public Architecture Boundary

- **Status:** Accepted
- **Decision owner:** William Hargis

## Context

Flip is a production application with implementation details that stay out of public view: production prompts, credentials, vendor pricing, and internal configuration. Reviewers and public audiences still need to understand what the system does and how it is structured.

## Decision

This repository will describe Flip as public architecture only. It will:

- document capabilities using structural names (module names, route shapes, service names, protocol names);
- publish diagrams, ADRs, and demo scenarios;
- link only to the live public hosts `https://flip.engineering` and `https://flip.tech-demo.dev`;
- omit source code, source mirrors, production persona prompts, secrets, credentials, vendor pricing, and internal URLs.

It will not:

- create a source mirror or a public git remote for Flip;
- quote implementation source beyond short structural names;
- publish internal email excerpts, reviewer/applicant names, or conflict findings.

## Consequences

- Reviewers can evaluate the architecture without accessing implementation internals.
- The overview cannot answer implementation-level questions about prompts, exact algorithms, or configurations.
- Maintaining the boundary requires ongoing discipline: every document in this repository must be checked against the public-architecture rule before publication.
