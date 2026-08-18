# 01 — Product and Problem

## Chat and forums solve opposite halves of community knowledge

Chat is effective because it is immediate, contextual, and socially low-friction. Its weakness appears later: search returns fragments, decisions detach from rationale, and important explanations remain buried in chronology.

Forums preserve durable, navigable discussion. Their weakness is activation energy. Participants do not naturally stop a live exchange, reconstruct its context, and publish a polished thread every time something useful happens.

Flip treats this as one lifecycle:

```text
live exchange
  -> topic, insight, or decision signal
  -> durable forum structure linked to the source
  -> later discovery and continuation
  -> new live exchange
```

Chat remains chat and forum remains forum. The product creates an explicit bridge rather than flattening both into one feed or a generated summary.

## The model participates in the social system, not above it

Flip has members, administrators, explicitly configured AI participants, and background workflows. Their authority differs.

A member can act within community membership and content rules. An administrator configures membership, moderation, feature access, curation, and AI behavior. An AI participant can answer or act only through capabilities admitted for its current surface and the invoking actor’s scope. A curation workflow can restructure eligible discussion but cannot impersonate the participants. Background jobs can retry work but cannot invent a product effect outside the domain transaction they owe.

These differences are product semantics, not prompt personalities.

## The central user journeys

### Preserve a valuable live discussion

Members debate an issue in chat. A curation run selects the relevant source set and proposes how it should appear in the forum. Flip validates the participant and message identities, destination, and duplicate state before committing the durable structure. The source room receives a linkback, and participants can correct or request recuration later.

The outcome is searchable knowledge with an inspectable path back to the conversation.

### Ask a current-information question

A member invokes an AI participant. The server derives the room and actor scope, selects bounded context, and admits external retrieval. The model discovers and reads sources; the product stores the selected evidence as citation records. The final answer is attributed to the AI and rendered with a source ledger.

The outcome is not “the model searched the web.” It is a durable product reply whose evidence can be inspected independently of the provider conversation.

### Ask about prior community knowledge

The same AI participant can retrieve chat or forum records the invoking actor is allowed to read. Authorization travels into the child tool task; the model cannot select another community in its JSON arguments. The answer links to product-native content and no valid scope means no protected results.

The outcome is internal knowledge assistance without a global cross-community search backdoor.

### Create or continue an artifact

A member may ask for a chart, document analysis, image, video, poll, or another product-native artifact. The AI can gather inputs and invoke a typed capability. Flip creates a durable request/artifact identity, represents pending or failed state, and can run one continuation when asynchronous provider work completes.

The outcome remains part of the conversation even when the producing model turn is long gone.

## Product invariants

### Authorship remains visible

Human source text remains attached to the human author. AI-authored replies remain attached to the AI identity. Structural curation text is distinguishable from both. A reader should not need an internal agent trace to know who produced what.

### Authorization follows the effect

Membership and visibility are checked by server code. Internal retrieval receives trusted actor/community scope. Product actions pass through domain services. A client or model cannot widen authority by changing arguments or local state.

### User-visible effects are durable

Replies, forum artifacts, citations, generated artifacts, and product actions have identities and lifecycle outside the model context. Retried background work converges on one intended effect when duplication would be harmful.

### Failure remains honest

Tool or provider failure does not authorize fabricated evidence. An invalid terminal reply is repaired or rejected before persistence. A failed asynchronous artifact remains visibly failed rather than being presented as completed. A transaction failure produces no phantom client update.

## What Flip deliberately is not

Flip is not a general autonomous agent detached from product context, a vector-database wrapper, a transcript summarizer that silently replaces member language, a client-authoritative offline database, a model-hosting platform, a long-horizon research truth-maintenance system, or a coding-agent fleet driver.

Those are distinct problems addressed, where relevant, by HomeCloud, Project Manager, and Baton. Keeping those boundaries explicit allows Flip to expand AI capability without losing its identity as a community product.