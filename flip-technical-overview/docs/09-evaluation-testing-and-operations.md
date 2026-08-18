# 09 — Quality and Evaluation Strategy

## A good model response is not evidence that the product works

Flip combines deterministic product rules with non-deterministic model behavior and asynchronous infrastructure. Its quality strategy therefore separates four questions:

1. Are the product invariants correct regardless of model?
2. Does the selected model route behave acceptably for its assigned surface?
3. Do the full user workflows converge across HTTP, jobs, database, sync, and UI?
4. Does deployed behavior remain healthy and correct over time?

<img src="../diagrams/ci-quality-gates.svg" alt="Flip quality and release evidence" width="900" />

## 1. Prove the deterministic shell

The highest-value tests target the code-owned contracts that must survive every model change:

- authorization scope cannot widen during internal retrieval;
- duplicate triggers do not produce duplicate visible effects;
- tool failure cannot silently kill an AI turn;
- curation cannot attribute text to the wrong participant or reference ineligible messages;
- invalid citation/artifact identities do not render as evidence;
- provider protocol cannot be persisted as normal content;
- a failed transaction cannot publish a phantom realtime success;
- optimistic clients converge on canonical identities;
- asynchronous continuation happens once for the intended terminal event.

These checks belong in domain, database, integration, job, adapter, and client tests. They should assert durable product state rather than merely that a function returned `:ok`.

## 2. Evaluate model behavior by surface

Model quality is not one benchmark score. The relevant behavior depends on the role.

For a research reply, the evaluation asks whether the model recognizes the need for current evidence, reads rather than cites search snippets, selects the right tools, handles disagreement, places citations near supported claims, and remains honest when evidence is insufficient.

For curation, it asks whether the plan forms coherent topics, preserves source identity and reply relationships, chooses a valid destination, avoids duplication, and defers material that needs review.

For action or artifact workflows, it asks whether the model uses typed capabilities correctly, produces a valid terminal response, and handles pending or failed external work without claiming success.

Evaluation cases are versioned with the route, model, adapter, configuration, and date. A result supports a deployment decision for that surface; it is not a timeless claim about the model.

## 3. Test convergence across the product

End-to-end scenarios cover the seams that unit tests cannot:

```text
user command
  -> authorization and transaction
  -> background job / model / tools
  -> durable reply, curation, or artifact state
  -> Electric / channel projection
  -> web or native rendering
```

The adversarial cases matter most: response and sync arriving in either order, disconnect during a pending mutation, provider failure after evidence collection, worker crash during tool dispatch, linkback failing after forum commit, permission revocation against cached state, or a client waking after an artifact completed.

The expected result is coherent state and honest failure, not one exact packet order.

## 4. Use operations as quality feedback

Runtime telemetry should explain where user-visible outcomes degrade without retaining unnecessary private content. Useful signals include queue wait, provider and tool latency, terminal reason, retry/repair rate, citation validation failure, curation correction, artifact abandonment, sync lag, and route-specific evaluation drift.

Product feedback matters as much as infrastructure health. Participant correction, recuration, deletion of AI replies, source-link traversal, and unresolved artifact state reveal failures that a latency dashboard cannot.

Operational data should feed new deterministic regressions or versioned evaluation cases rather than becoming a pile of unexamined logs.

## Citation quality has deterministic and semantic layers

Flip can deterministically check that a citation identity exists, the source was retrieved, and the selected passage appears in it. It cannot deterministically prove that the passage is true, current, or sufficient for every nearby inference.

Semantic evaluation therefore considers relevance, claim scope, source type/date, and disagreement. High-stakes or disputed cases may require human review. The architecture is explicit about where automation ends.

## Security regressions are product regressions

The suite must exercise cross-community retrieval denial, forged scope arguments, missing trusted scope, private source leakage into public artifacts, unsafe external fetching, prompt injection in retrieved content, rendered-content sanitization, unauthorized product actions, stale client capability, and credential redaction.

These are not separate from AI quality. An answer that is relevant but violates visibility is a failed product outcome.

## CI and release evidence

A practical deterministic gate compiles and formats the server, applies migrations, runs server/client/integration/security tests, checks dependencies and documentation, and builds the release/container. Selected end-to-end scenarios verify the assembled product.

Model/provider evaluations are slower and can run on scheduled or pre-release tracks, but route changes should not ship without current evidence for the surfaces they affect. When a route regresses, the product can constrain or remove that route without blocking unrelated deterministic changes.

## Failure-domain evidence

The test and operations model should confirm that:

- model or tool outages degrade AI capability while ordinary collaboration remains coherent;
- queue backlog delays durable work without inventing completion;
- sync/channel interruption preserves PostgreSQL authority;
- media failure produces a failed artifact rather than a broken conversation;
- BEAM process restarts do not corrupt durable state;
- database unavailability prevents non-durable success from being published.

## Evidence discipline

Claims such as “production-ready,” “zero hallucinations,” “supports N users,” or “novel” require dated, reproducible evidence. The portfolio instead states the contracts, evaluation methods, failure boundaries, and known limitations that can be defended from the implementation.