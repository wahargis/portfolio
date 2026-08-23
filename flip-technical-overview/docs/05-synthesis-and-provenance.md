# Curation and provenance

Flip supports direct AI-authored content and curation of selected human conversation. These operations have different authorship, source, access, and correction requirements and therefore use separate workflows.

![Chat-to-forum curation](../diagrams/synthesis-pipeline.svg)

## Direct AI authorship

A direct AI reply is new content attributed to an AI identity. The associated activity records the invoking actor, origin, route, tools, sources, artifacts, and terminal outcome. The reply can include citations or generated artifacts, but its authorship remains the AI identity.

Direct authorship is appropriate for answers, analysis, generated descriptions, product assistance, and other content that did not previously exist as selected participant messages.

## Conversation curation

Curation turns selected conversation into durable forum structure. The source content remains attributable to the original participants.

A curation run can:

1. identify the source room and selected messages;
2. load the messages under current actor and source visibility;
3. classify or group the material into topics;
4. propose destination forums, threads, and ordering;
5. validate destinations and publication rules;
6. create or update forum objects;
7. attach participant and source-message relationships;
8. create linkback to the source conversation;
9. record run, feedback, and repair state.

Generated headings or bridge text can be stored as AI-authored additions when allowed. They do not replace the authorship of selected messages.

## Durable run state

Curation is represented as a multi-stage run rather than one prompt call. State can include:

- requesting actor and source room;
- selected messages and selection version;
- target community and allowed destinations;
- current stage and worker ownership;
- proposed topic and destination plan;
- validation errors and operator decisions;
- created or updated forum objects;
- participant and message provenance;
- linkback and notification status;
- feedback, correction, and recuration history;
- retry, stale-work, failure, and completion state.

A run identity prevents duplicate publication and supports repair after process interruption.

## Source selection

Selection can be explicit or produced by a product workflow. The server records the selected message identities and validates that the requesting actor can read them when the run is admitted and again before publication.

The run operates on the stored selection rather than a later unbounded room transcript. If the source changes, the system can record a new selection or recuration version instead of silently changing the evidence used by an existing publication.

## Planning and validation

A model can propose topics, titles, summaries, ordering, and destinations. The application validates the proposal against:

- available communities, forums, and thread types;
- current actor and source access;
- allowed creation and update operations;
- source-message coverage and duplication;
- title and content constraints;
- provenance requirements;
- current state of existing destination objects.

Invalid or stale destinations are resolved or rejected before forum mutation. The model does not receive direct authority to create arbitrary forum structure.

## Publication transaction

Forum objects and their source relationships should commit in a transaction appropriate to the operation. A newly created thread should not become visible without the provenance records required by the product.

Later stages such as source-room linkback, notifications, or optional media can commit separately. Their state remains attached to the run so failure can be repaired without repeating the completed forum mutation.

## Origin-aware access

A curation-derived forum object retains a relationship to its source messages and room. Access resolution considers both the destination and the source boundary.

A broader destination does not automatically make restricted source content public. The most restrictive applicable boundary remains effective unless a separate authorized workflow creates a new publication with appropriate review and provenance.

Search, direct reads, AI retrieval, and client synchronization use this effective access rather than only the destination forum's default visibility.

## Feedback and correction

Users and operators need correction paths because topic grouping, destination choice, bridge text, and message selection can be wrong even when the workflow completes technically.

Feedback can be attached to the run or published object. A correction can update generated framing, adjust destination or organization, add or remove selected material, or start a bounded recuration from an explicit source version.

Corrections preserve earlier run and publication history. The system should not erase which source selection and model result produced the earlier state.

## Partial failure

Curation contains several effects that can fail independently:

| Failure | Recovery behavior |
|---|---|
| Source load or authorization failure | Stop before planning or publication and retain the failed run. |
| Invalid model plan | Repair or rerun planning without creating forum objects. |
| Forum mutation failure | Retry under the same run and operation identities where safe. |
| Provenance commit failure | Fail the transaction so a publication is not left without required source relationships. |
| Linkback or notification failure | Keep the completed forum result and retry only the failed later stage. |
| Worker crash | Recover the run from its last durable stage and ownership state. |
| Duplicate request | Return or continue the existing run rather than publish again. |
| Source visibility changes | Recheck access before reading or publishing and stop when the original scope is no longer valid. |

## Provenance queries

The stored relationships support questions such as:

- Which messages and participants produced this forum item?
- Which AI-generated headings or bridge text were added?
- Which curation run and selection version created the current publication?
- Was the source room more restrictive than the destination?
- Which feedback or recuration changed the result?
- Which publication objects were affected by a failed or retried run?

These are product queries over durable state rather than explanations reconstructed from a provider transcript.

[Previous: Retrieval, evidence, and tools](04-retrieval-search-and-tools.md) · [Next: Data, realtime, and clients](06-data-realtime-and-clients.md)
