# Community and forum data flow

Flip uses the same identity and membership model across chat, forums, AI curation, moderation, and live rooms. Community content remains authored user content even when an AI service reads it or produces a separate summary.

## Message and post creation

A client submits a chat message or forum post with the target community or conversation. The server:

1. Resolves the authenticated user.
2. Checks membership, block, moderation, and target visibility rules.
3. Stores the authored record in PostgreSQL.
4. Publishes the durable change to connected clients.
5. Emits transient channel events for immediate interface feedback.
6. Starts AI follow-up work only when the community or user settings allow it.

The stored message or post is the source record. Delivery acknowledgements, typing state, presence, and progress notifications are separate transient events.

## Edits and deletion

Edits preserve the identity of the source record. Deletion uses stored state so clients, citations, moderation tools, and AI follow-up work can respond consistently. Message edit and delete tracking can schedule refresh work for generated outputs that depended on the changed source.

A generated summary does not become a replacement for the discussion. It has its own author type, lifecycle, and citation set.

## Community AI replies

Community AI operates under the requesting user's visibility and the community configuration. The runtime builds a request that contains:

- The target conversation or thread.
- Visible source messages or posts.
- Participant identity and role information required by the feature.
- Community instructions and enabled capabilities.
- A server-selected model and provider route.
- Tool permissions that apply to the request.

The worker does not receive unrestricted application access. Any tool call returns to a server handler that checks the request identity, target resource, and tool policy.

## Forum synthesis and citation

Forum synthesis converts a source set into a stored synthesis record. Citation records connect output spans or claims to the source post and author identifiers. This supports:

- Opening the source material from the generated output.
- Showing who wrote the source.
- Rebuilding or marking a synthesis when a cited source changes.
- Keeping generated authorship separate from user authorship.
- Recording which records were available to the synthesis job.

Synthesis is background work. The initial request and source selection are stored before model execution. Completion writes the output and citation records in one application workflow.

## Real-time delivery

Flip uses two delivery paths:

- PostgreSQL-backed synchronization carries durable records such as messages, posts, replies, and library items.
- Phoenix Channels, PubSub, and Presence carry transient signals such as typing, participant presence, progress, and media-room control events.

The client can receive a fast transient indication and later confirm the durable record through synchronization. The server database remains the authority when the two paths arrive at different times.

## Failure handling

A failed AI reply or synthesis does not change the source discussion. The job records a retryable or terminal state. Reconciliation work can find records that were queued or running when a worker stopped.

Source edits and deletion are handled as new state changes. The system does not depend on reconstructing the original browser request to determine which content changed.
