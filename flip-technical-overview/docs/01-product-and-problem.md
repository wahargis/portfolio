# 01 — Product and Problem

## Why a combined chat and forum product exists

Chat optimizes for immediacy. A message is useful because participants share recent context, social awareness, and a live objective. Its weakness appears later: search returns fragments, decisions are detached from rationale, and valuable explanations remain buried in chronology.

Forums optimize for durable structure. A thread can be found, linked, ranked, revisited, and extended. Its weakness is activation energy: participants do not naturally restate every useful live exchange as a polished post.

Flip treats this as one lifecycle:

```text
live exchange
  -> topic and decision signal
  -> durable structure
  -> later discovery and continuation
  -> new live exchange
```

The product does not force every message through the same representation. Chat remains chat; forum content remains forum content; synthesis creates explicit links between them.

## Product actors

| Actor | Responsibilities and capabilities |
|---|---|
| **Member** | Participate in rooms and forums, search content, react, vote, bookmark, create artifacts, invoke permitted AI assistance. |
| **Room/community administrator** | Configure membership, moderation, synthesis eligibility, AI participant behavior, feature access, and product settings. |
| **AI participant** | Respond under an explicit identity, use admitted tools, create attributed content/artifacts, and operate within actor/community scope. |
| **Curator workflow** | Organize eligible conversation into forum structure while preserving source authorship and provenance. |
| **Background worker** | Execute durable asynchronous jobs with retries, uniqueness constraints, and explicit terminal state. |
| **Client** | Present and optimistically interact with state; reconcile against server authority. |

The AI participant is not an administrator and the curator workflow is not an author proxy.

## Capability domains

### Chat

Chat is the low-latency collaboration surface:

- rooms and memberships;
- messages, replies, reactions, pins, and read state;
- typing and presence;
- uploads and inline media;
- custom emoji packs and GIF favorites;
- full-text search;
- per-room synthesis and AI configuration;
- notifications and activity summaries.

### Forum

Forum is the durable discussion surface:

- communities/subforums and topic organization;
- threads and nested replies;
- voting, bookmarks, tags, and sort modes;
- source/origin metadata;
- links from synthesized material to chat;
- direct and AI-authored forum participation;
- enrichment and source ledgers.

### Synthesis

Synthesis is a curation pipeline, not generic summarization:

- select eligible source messages;
- partition material into coherent topics;
- map topics to durable forum placement;
- preserve quoted or copied user words and participant identity;
- generate bounded bridge/structural context;
- merge or deduplicate related output;
- create source linkbacks;
- accept structured feedback;
- rerun or escalate when automated curation is insufficient.

The workflow must remain inspectable: readers should be able to move from the durable artifact back to the source conversation.

### AI participation

An AI participant can:

- answer a direct mention or reply;
- search current external information;
- retrieve authorized internal discussion;
- read webpages and documents;
- compare sources and create citations;
- query structured data;
- render charts or rich data;
- create/edit media artifacts;
- draft polls or perform selected platform actions;
- continue after asynchronous artifact completion;
- participate in constrained specialized surfaces such as a game room.

The available subset is computed from context. Listing a capability in the platform does not imply every user or room can invoke it.

## Primary user journeys

### From live debate to durable record

1. Members debate an issue in chat.
2. The curation workflow identifies a coherent topic and source set.
3. It creates or enriches a forum thread.
4. Source relationships and linkbacks are persisted.
5. Participants review, correct, or request recuration.
6. Later readers find the thread and can inspect the originating exchange.

### Current-information question

1. A member invokes an AI participant in a room.
2. The runtime derives the actor/community scope.
3. The model chooses external search and source-reading tools.
4. The server executes the tools and mints evidence records.
5. The model composes an attributed answer with citation tokens.
6. The server validates/persists the reply and source ledger.

### Internal knowledge question

1. A member asks about a prior room or forum discussion.
2. Internal search tools receive the origin actor and community.
3. Authorization-constrained retrieval returns only readable records.
4. Neighboring context is supplied to the model.
5. The answer links to product-native content rather than inventing an external source.
6. Missing scope yields no results and an explicit model-visible warning.

### Artifact request

1. A member requests a chart, image, document analysis, or video workflow.
2. The AI gathers inputs and invokes the relevant typed tool.
3. A durable artifact/request record is created.
4. Immediate or asynchronous provider work completes.
5. The result is attached to the conversation with provenance.
6. A continuation turn can interpret the terminal result and communicate next steps.

## Product invariants

### Authorship

- Human-authored source content remains attributed to the human.
- AI-authored content remains attributed to the AI identity.
- Curation-generated structural text is distinguishable from source messages.
- A forum artifact can be inspected without implying that the AI wrote the participants’ words.

### Authorization

- Membership and content visibility are checked by server-side code.
- Internal retrieval cannot become a cross-community search shortcut.
- Child tool tasks receive the same origin scope as the parent reply.
- Product actions pass through domain authorization rather than direct database writes from the model.

### Durability

- A user-visible effect has a durable state record.
- Retries converge on one intended effect when duplicate output would be harmful.
- Source/citation/artifact identities survive beyond one model context window.
- Client state can be rebuilt from server truth.

### Honest failure

- Tool crashes become bounded, model-visible failure results.
- Provider failures do not authorize fabricated evidence.
- A failed asynchronous artifact remains distinguishable from a completed result.
- Terminal recovery can disclose failure without presenting infrastructure text as model-authored research.

## Non-goals

Flip is not:

- a general-purpose autonomous agent operating without product context;
- a vector-database wrapper presented as a community product;
- a transcript summarizer that silently replaces user language;
- a client-authoritative offline database;
- a model-hosting platform—local inference is supplied by a separate system such as HomeCloud;
- a long-horizon research truth-maintenance system—Project Manager owns that problem;
- a coding-agent control plane—Baton owns that problem.

These non-goals keep the product model coherent even as AI capabilities expand.
