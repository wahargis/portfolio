# Flip architecture scenarios

The synthetic technical environment is intended to make architectural behavior inspectable without production data. These scenarios focus on product contracts rather than scripted reviewer choreography.

Product reference: <https://flip.engineering>  
Synthetic technical environment: <https://flip.tech-demo.dev>

Endpoint availability is not required to use the case study; the expected state transitions are documented below.

## Scenario 1 — Current-information answer with citations

### Request

Ask an AI participant a question that requires current external evidence and at least two sources.

### Inspect

1. The user invokes an explicit AI identity.
2. The runtime exposes search and source-reading tools.
3. Search results are followed by direct page reads.
4. Source records/citations are minted before the final answer.
5. The final reply contains rendered citation links or a source ledger.
6. Tool/provider failure is described honestly rather than replaced by fabricated evidence.

### Architecture pages

- [Agent Runtime](../docs/03-agent-runtime.md)
- [Retrieval, Search, and Tools](../docs/04-retrieval-search-and-tools.md)
- [Model Routing and Inference](../docs/07-model-routing-and-inference.md)

## Scenario 2 — Authorization-scoped internal retrieval

### Request

Ask about prior discussion in a room or forum the current user can read. Compare with a room the user cannot read, where the synthetic role model permits this test.

### Inspect

1. The worker derives actor and origin-community scope from product state.
2. Internal search receives trusted scope outside model arguments.
3. Readable content is returned with product-native identifiers.
4. Missing or denied scope produces no cross-community results.
5. The final answer links to accessible content only.

### Architecture pages

- [Product and Problem](../docs/01-product-and-problem.md)
- [Retrieval, Search, and Tools](../docs/04-retrieval-search-and-tools.md)
- [Quality, Evaluation, and Operations](../docs/09-evaluation-testing-and-operations.md)

## Scenario 3 — Chat-to-forum curation

### Request

Use a seeded chat exchange containing a coherent decision or explanation and invoke or observe its curation into forum structure.

### Inspect

1. The source messages and participants remain identifiable.
2. The durable forum result is attributed as sourced/curated, not presented as if the AI authored the participants’ statements.
3. Structural bridge text is distinguishable from source content.
4. The forum artifact links back to the originating chat.
5. Feedback or recuration state is visible where the scenario includes it.

### Architecture pages

- [Synthesis and Provenance](../docs/05-synthesis-and-provenance.md)
- [System Architecture](../docs/02-system-architecture.md)

## Scenario 4 — Structured data or artifact

### Request

Ask for a chart, rich-data view, document analysis, or configured media artifact.

### Inspect

1. The AI invokes a typed tool rather than embedding an invented artifact in prose.
2. The product creates a durable artifact/request identity.
3. The UI distinguishes pending, completed, and failed state.
4. A completed asynchronous result can trigger one continuation turn.
5. Inputs, source records, and the conversation attachment remain linked.

### Architecture pages

- [Retrieval, Search, and Tools](../docs/04-retrieval-search-and-tools.md)
- [Agent Runtime](../docs/03-agent-runtime.md)
- [Data, Realtime, and Clients](../docs/06-data-realtime-and-clients.md)

## Scenario 5 — Realtime convergence

### Request

Open the same synthetic room or forum in two client sessions, then create a message/reply or reaction.

### Inspect

1. The initiating client may display optimistic state.
2. The server authorizes and commits the mutation.
3. Durable synchronization delivers the canonical record.
4. The initiating client reconciles rather than duplicating it.
5. Ephemeral typing/presence remains separate from durable message state.
6. Reconnect rebuilds from server truth.

### Architecture pages

- [Data, Realtime, and Clients](../docs/06-data-realtime-and-clients.md)
- [Quality, Evaluation, and Operations](../docs/09-evaluation-testing-and-operations.md)

## Scenario 6 — Controlled failure

A technically credible demo should include failure state.

Possible synthetic conditions:

- external source unavailable;
- one tool times out;
- model endpoint unavailable;
- artifact provider returns terminal failure;
- invalid citation/output is rejected;
- curation linkback fails after forum creation.

Inspect whether the product preserves the durable state that did succeed, labels the failed stage, avoids duplicate effects, and gives the user an honest next state.

## Expected evidence

A scenario is convincing when a reviewer can inspect:

- the durable product object;
- authorship and source relationships;
- citation/artifact identities;
- visible lifecycle state;
- actor/community authorization outcome;
- realtime convergence;
- a failure or correction path.

A polished model answer alone is not sufficient evidence of the architecture.
