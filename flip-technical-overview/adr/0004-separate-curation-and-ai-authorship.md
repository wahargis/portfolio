# ADR 0004: Separate curation from direct AI authorship

- **Status:** Accepted
- **Scope:** AI-created and AI-assisted content

## Context

A direct AI reply creates new content attributed to an AI identity. Conversation curation selects and reorganizes content written by human participants. Treating both as generic generation would make authorship, source relationships, access, and correction unclear.

## Decision

Use separate workflows and data relationships:

- direct AI content is attributed to the AI identity and associated with its activity, tools, sources, and artifacts;
- curation records selected messages, source participants, plan and destination state, forum objects, linkback, feedback, and recuration history;
- generated headings or bridge text in curation are identified as AI additions rather than reassigned source authorship;
- source visibility remains part of access decisions for curation-derived content.

## Consequences

The application maintains two related but distinct AI content paths. Curation requires more durable state than a summary call, including selection version, destination validation, source relationships, partial failure, and correction.

The product can answer who wrote a piece of content, which conversation produced a forum item, and which generated additions were made during curation.

## Revision conditions

Add another content mode only when its authorship and provenance requirements cannot be represented by direct AI authorship or conversation curation.
