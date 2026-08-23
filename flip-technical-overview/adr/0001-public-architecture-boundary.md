# ADR 0001: Private implementation and public architecture portfolio

- **Status:** Accepted
- **Scope:** Public technical material for Flip

## Context

Flip is implemented in a private repository and operates with product data, credentials, provider integrations, deployment configuration, prompt and persona state, and administrative controls that are not part of a public portfolio.

Technical review still requires enough information to understand the product architecture, agent runtime, data and authorization boundaries, failure handling, client behavior, and engineering decisions. Linking to an unavailable private source repository is misleading and does not provide useful public evidence.

## Decision

Keep the implementation repository private. Publish a separate technical portfolio containing:

- project and system descriptions;
- execution and state diagrams;
- architecture decision records;
- status and limitation documentation;
- synthetic technical scenarios;
- local links between the public portfolio pages.

Do not link the public portfolio to private implementation repository paths. Do not include product data, user content, credentials, provider keys, private deployment state, host identifiers, prompt and persona state, or administrative endpoints.

## Consequences

The public material must explain the system without assuming source access. Diagrams and prose need enough technical detail to show real execution, state ownership, and failure handling rather than only naming components.

The portfolio can describe implementation classes and technologies at the system level, but code-level claims that require private inspection should be presented as architecture and current behavior, not as publicly verifiable source links.

Changes to the private system that affect public contracts require corresponding updates to the portfolio, diagrams, decisions, and synthetic scenarios.

## Verification

A documentation audit should check for:

- links to private Flip source paths;
- copied product identifiers, data, or screenshots containing sensitive content;
- credentials, tokens, provider keys, and internal hostnames;
- environment-specific deployment details that grant or reveal authority;
- claims that the private implementation is publicly available;
- diagrams that imply access or integrations not represented by the public material.

## Revision conditions

Reconsider this decision only if the source-availability policy changes. Any source publication requires a separate review of data, secrets, licensing, operational authority, generated assets, and deployment-specific configuration.
