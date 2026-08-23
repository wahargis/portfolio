# ADR 0008: Durable citations and artifacts

- **Status:** Accepted
- **Scope:** Evidence, files, charts, generated media, and other agent results

## Context

A model reply can refer to web sources, product records, documents, tables, charts, files, images, videos, polls, and other results. Provider text alone cannot establish that these objects exist, completed successfully, remain visible to the user, or can be used by later workflows.

Long-running media and document work can also finish after the original model turn.

## Decision

Store sources, citations, files, charts, generated media, polls, and other artifacts as application objects with stable identities, access state, lifecycle, and relationship to the AI activity or product object that created them.

Validate terminal references against stored sources and artifacts. Long-running operations create pending objects before provider completion and can start one deduplicated continuation after terminal state.

## Consequences

The application manages artifact schemas, storage, cleanup, access, retries, provider attempts, and client rendering. Replies can show failure or pending state instead of presenting an invented completed result.

Later users and workflows can inspect or reuse durable evidence and artifacts without replaying the original provider conversation.

## Revision conditions

New result types can use different storage or lifecycle systems, but they must expose stable application identities, access behavior, and terminal state to the agent runtime and clients.
