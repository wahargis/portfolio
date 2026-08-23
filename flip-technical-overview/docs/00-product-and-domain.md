# Flip Product and Domain Model

Flip is a community product that combines live chat, durable forums, media, search, and explicit AI participation. Its principal product problem is not message transport. It is preserving the meaning, audience, authorship, and lifecycle of content as it moves between immediate conversation, model-assisted work, asynchronous processing, and long-lived publication.

## Product surfaces

### Chat

Chat supports rooms, messages, replies, reactions, polls, media, membership, presence, typing, read state, and search. It is optimized for immediate participation and conversational continuity.

AI replies enter this surface as visible messages from an identifiable system participant. A user can see what interaction triggered the reply and can continue the thread through normal chat behavior.

### Forum

Forums support durable threads, nested replies, voting, tags, bookmarks, moderation, search, and origin metadata. They are optimized for content that should remain navigable after the live conversation has ended.

A forum thread may be authored directly or produced through a synthesis run. Those origins remain distinguishable in the data model.

### Synthesis

Synthesis is the workflow that selects authorized chat content and creates a durable forum artifact. It carries its own run identity, source selection, destination, status, retries, result, and feedback.

Synthesis does not redefine chat as a temporary draft of a forum. Most conversation remains conversation. Durable publication is an explicit product transition.

### AI participation

The AI participant runtime handles explicit conversational turns, product tools, retrieval, citations, generated artifacts, and asynchronous continuation. It is part of the community application and uses the same identity, authorization, persistence, and moderation boundaries as human participation.

## Principal domain state

| Domain | Representative state |
|---|---|
| **Accounts** | Users, sessions, profiles, roles, memberships, and identity used by every product surface. |
| **Communities** | Servers or groups, membership, moderation roles, feature configuration, and audience boundaries. |
| **Chat** | Rooms, messages, reply ancestry, reactions, polls, pins, files, search state, and AI triggers. |
| **Forum** | Subforums, threads, replies, votes, bookmarks, tags, publication status, and source relationships. |
| **Synthesis** | Runs, claim state, selected messages, participants, destination, retry state, feedback, and result. |
| **AI activity** | Model turn, provider route, tool calls, terminal status, citations, warnings, and audit data. |
| **Artifacts** | Generated or transformed files with type, lifecycle, preview, source activity, and delivery state. |
| **Capabilities** | Deployment and client feature availability for authentication, sync, uploads, push, deep links, and AI operations. |

## Product invariants

### AI turns have a visible cause

A mention, reply, or explicit product action creates the turn. The system does not treat all room traffic as ambient model input.

### Context follows actor and room scope

The reply path revalidates the triggering actor and loads only eligible room state. Same-room reply ancestry can preserve conversational continuity; cross-room recall is not inferred from database availability.

### Curation and authorship remain distinct

A synthesized thread organizes existing human conversation. An AI reply creates new AI-authored content. Their provenance, presentation, and moderation semantics remain separate.

### Source restrictions survive publication

A forum destination does not erase the audience of the source room. A synthesized artifact derived from restricted conversation cannot become readable to a broader audience solely because the forum has broader default visibility.

### Durable work has explicit state

Synthesis runs, AI activities, tool calls, citations, and artifacts do not exist only as logs. They have identities and statuses that can be queried after the request or model context has ended.

### Client capability is negotiated

Web, desktop, mobile, and deployment variants do not assume that every feature is configured. Capability metadata determines whether the client exposes authentication, push, upload, synchronization, and other optional behavior.

## Product workflows

### Conversational AI turn

```text
visible trigger
  -> actor and room validation
  -> bounded context
  -> capability and effect admission
  -> durable model/tool loop
  -> activity, citations, and artifacts
  -> AI-authored chat message
```

### Conversation curation

```text
authorized source messages
  -> synthesis run and claim
  -> source and topic selection
  -> forum thread creation
  -> source relationships and audience enforcement
  -> feedback or re-curation
```

### Long-running artifact

```text
AI tool call
  -> pending artifact and durable job identity
  -> provider or local media work
  -> completed or failed terminal record
  -> preview/update or bounded continuation turn
```

## Product architecture boundary

Flip owns community identity, access, context, tool policy, content state, and publication. Model providers supply inference and media capability through product-owned adapters. A provider failure can interrupt an AI turn, but it does not grant access to additional community data or change ordinary chat and forum authority.

The public portfolio documents this architecture and selected source paths. Production data, private implementation code, prompts, credentials, and security-sensitive configuration remain outside the public repository.

[← Flip case study](../README.md)
