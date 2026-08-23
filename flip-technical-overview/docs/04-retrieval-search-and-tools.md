# Retrieval, evidence, and tools

Flip's agent runtime uses application capabilities for product retrieval, external research, documents, data, media, and product effects. The server selects the capabilities available to each turn and applies trusted product scope before execution.

![Governed tool execution](../diagrams/governed-tool-execution.svg)

## Capability classes

| Class | Examples | Main controls |
|---|---|---|
| **Product reads** | Messages, rooms, forums, posts, search, members, polls, files, and existing artifacts. | Actor and community scope, object visibility, bounded result size, stable product references. |
| **External research** | Web search, direct page reads, document retrieval, source extraction, and evidence assembly. | URL and source validation, deadlines, content limits, source identity, citation tracking. |
| **Document and data analysis** | PDF and document reading, selected passages, tables, calculations, statistics, and charts. | File and dataset identity, typed inputs and outputs, size limits, reproducibility, artifact storage. |
| **Product effects** | Replies, polls, forum changes, attachments, generated files, notifications, and other domain operations. | Target authorization, domain validation, idempotency, effect identity, durable commit, user-visible state. |
| **Multimodal analysis** | Image and video description, OCR, frame or asset analysis. | Artifact visibility, modality route, size and duration limits, result identity. |
| **Long-running generation** | Image and video generation or editing, document production, and staged media workflows. | Pending artifact, provider attempt, progress, cancellation, retry, terminal state, continuation. |

## Capability admission

The tool list is created for one turn using:

- the product surface and requested workload;
- invoking actor and current membership;
- community and origin object;
- selected AI identity and feature configuration;
- route support for tools and modalities;
- configured services and current health;
- policy for effectful or long-running operations;
- working context and deadline budgets.

A tool that is not admitted cannot be invoked by name through another generic endpoint. The runtime does not provide the model with an unrestricted internal HTTP client, database connection, or file-system root.

## Trusted scope

Tool schemas separate user-controlled arguments from server-controlled scope.

A model may supply a search query, document question, poll options, or requested transformation. The runtime supplies the actor, community, origin object, AI identity, workspace root, and other trusted values required by the operation.

Tools reject attempts to override or broaden these values. Protected reads perform current authorization rather than assuming that context loaded earlier in the turn remains valid.

## Internal retrieval

Internal retrieval can combine product search, typed filters, direct object reads, and relationship traversal. The result includes stable product identities and enough metadata to support later authorization and citation.

Internal retrieval should answer questions such as:

- Which messages in the current room discuss the requested topic?
- Which forum items visible to this actor provide durable background?
- Which artifact or document is attached to the referenced message?
- Which product object is the source of a synthesized forum result?
- Which prior AI activity created the artifact used in this turn?

The search index and retrieval layer do not create a new permission system. A search hit is returned only when the underlying object is currently readable.

## External research

![Retrieval, source discovery, and citation](../diagrams/retrieval-source-citation-flow.svg)

External research separates source discovery from evidence reading.

```text
search query
  -> candidate results and metadata
  -> direct read of selected pages or documents
  -> stored source and passage records
  -> analysis or synthesis
  -> final citations validated against stored evidence
```

Search snippets can guide selection but are not automatically accepted as full evidence. Direct reads can fail, return incomplete content, or provide unsupported material. Those outcomes remain visible to the runtime and can affect the final response.

Source records can include canonical URL, title, retrieval time, content type, selected passages, tool status, and relationship to the AI activity. The reader-facing reply refers to the sources actually used rather than a list invented after generation.

## Documents and PDFs

Document operations use a durable file or source identity. The system can extract or select passages, answer questions, produce structured results, or create a derived artifact.

Large documents are processed in bounded segments or through an index appropriate to the workload. The final answer can retain passage or page references where available. Parsing, OCR, or extraction failures are stored separately from the model's interpretation.

A document visible to one actor is not placed in a globally accessible agent memory. Later access checks still apply to the durable file and derived result.

## Structured data and charts

Data tools use typed requests and results. A model can select a dataset, columns, filters, aggregation, or analysis operation, but the application or analysis runtime performs the calculation.

Stored results can include:

- the source dataset and version;
- query or transformation parameters;
- table schema and rows used by the result;
- statistics or model output;
- chart specification and rendered artifact;
- execution status and errors.

This allows a later user or system to inspect the data operation without treating generated prose as the calculation record.

## Product effects

A product effect follows the owning domain service.

For example, a poll request must identify an allowed target, pass option and lifecycle validation, receive a durable poll identity, and create the same product events expected by clients. A forum action must pass forum and source visibility rules. A generated file must receive an artifact identity and storage state before a reply can refer to it.

Effect calls use idempotency or operation identities where duplicate execution would create user-visible damage. A provider retry does not automatically repeat a committed effect.

## Long-running tools

Long-running generation creates a pending artifact or job and returns that object to the turn. Provider execution can then continue asynchronously.

The durable state includes:

- requested operation and selected inputs;
- model or provider route;
- provider attempt identity;
- progress and status;
- output artifact or failure;
- cancellation and retry state;
- parent activity and continuation identity.

A completed artifact can be used by later product flows without requiring the provider session that created it.

## Citation and artifact validation

Before a terminal reply is stored, the runtime can check that referenced sources and artifacts exist, belong to the current activity or admitted context, and are visible to the target audience.

Invalid references can be removed, repaired, or cause the terminal response to fail according to the product surface. The system does not publish a citation merely because the model emitted a plausible identifier or URL.

## Failure handling

- Unknown or unadmitted tools are rejected.
- Schema-invalid calls do not reach product services.
- Protected reads fail when the actor or origin scope is missing or stale.
- Tool timeouts and provider errors remain distinct from successful empty results.
- Effectful calls record whether an effect was committed before any retry.
- Long-running calls preserve pending, failed, cancelled, and completed artifact state.
- Source-read failure does not become a supported citation.
- Artifact cleanup respects retained product references and lifecycle state.

[Previous: Agent runtime](03-agent-runtime.md) · [Next: Curation and provenance](05-synthesis-and-provenance.md)
