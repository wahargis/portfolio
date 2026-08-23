# Flip Retrieval, Tools, and Artifacts

Flip exposes AI capabilities through typed product operations. Retrieval, document work, media generation, polls, games, and other effects are admitted for a specific actor and surface, executed through application code, and recorded independently from the model response that requested them.

## Capability construction

The tool set for a turn is assembled at runtime. It can depend on:

- the triggering actor and community role;
- the room or product surface;
- feature and room configuration;
- the selected model route and its tool protocol;
- deployment capability and provider health;
- required effect authority;
- available time, context, and infrastructure capacity.

This prevents the tool registry from becoming an unrestricted global menu. A tool may exist in the codebase without being available in ordinary chat, to a particular actor, through a route that lacks the required schema behavior, or when the backing service is unavailable.

## Tool contract

A product tool has more structure than a prompt description.

| Contract element | Purpose |
|---|---|
| **Name and schema** | Gives the model a stable typed operation and validates proposed arguments. |
| **Surface and actor scope** | Identifies where the operation may be offered and whose authority it uses. |
| **Effect classification** | Distinguishes reads, reversible updates, artifact creation, publication, and other consequential actions. |
| **Execution timeout** | Prevents one operation from consuming the complete conversational deadline. |
| **Normalized result** | Returns a stable shape to the model and application regardless of backend details. |
| **Audit projection** | Stores safe arguments, result classification, timing, and error state separately from the user-facing response. |
| **Artifact or citation output** | Mints durable product records when the result must remain usable after the turn. |

Tool parsing and argument sanitization occur before dispatch. Effect authority is checked after parsing; a syntactically valid call may still be denied because the current actor or product surface lacks permission.

## Retrieval paths

Retrieval is divided by source and authority rather than presented as one generic search function.

### Community retrieval

Internal retrieval can search or load authorized chat and forum state. The operation preserves product scope and object identity so retrieved content can be cited, linked, or checked against current access.

The model does not receive an unrestricted database query interface. Search and fetch operations go through domain functions that know room membership, forum policy, moderation state, source restrictions, and deleted content.

### External retrieval

External search and page retrieval can gather current public information through configured providers. Search results, selected pages, extraction status, and source metadata are returned as evidence rather than pasted into an untracked prompt buffer.

Provider failure, unavailable pages, extraction errors, and incomplete evidence remain visible to the worker. The final answer can distinguish a supported claim from a requested lookup that did not complete.

### Document retrieval

Documents are represented as application objects with ownership, access, file identity, extracted content, and processing state. A tool can search or read an authorized document without treating every uploaded file as globally available context.

Large documents can be read in bounded sections. PDF and structured-document operations can retain page, section, or object references required for citations and later inspection.

### Structured application state

Some questions are answered from typed product state rather than text retrieval. Poll results, game state, artifact status, room settings, or workflow records should be loaded through their owning domain and returned in a structured result.

## Evidence and citation lifecycle

A retrieval result becomes useful evidence only when its identity and relationship to the answer survive the model turn.

### Source records

The application records source type, location or product object, retrieval status, title or label, and other metadata needed to resolve the source later. Sensitive internal identifiers can remain in the administrative record while the public projection exposes a safe reference.

### Citation minting

The worker mints durable citation identifiers for sources it intends to use. The terminal response embeds those identifiers adjacent to the supported claims. A citation validator checks that referenced identifiers exist and are eligible for the reader.

### Source ledger

The source ledger projects the citations and relevant execution evidence associated with the final AI message. It can show which sources were used, whether a tool failed, or whether an artifact remains pending without exposing the complete internal prompt, hidden provider fields, or private tool arguments.

### Access after publication

For internal community sources, citation resolution still uses current authorization. A user who cannot read the source room should not gain access because an AI message contains a source token.

## Artifact model

Artifacts are typed durable outputs produced or transformed by an AI tool. Examples include images, video, audio, charts, files, documents, and structured data intended for rich rendering.

A typical artifact record carries:

- artifact type and media metadata;
- owning AI activity or product object;
- source tool call and request identity;
- pending, processing, completed, failed, or cancelled state;
- provider or local job identity;
- preview, download, or rendering references;
- terminal facts and safe error information;
- continuation and chain identity where several steps are required.

The chat message can refer to an artifact while its job continues. The artifact state, not the continued existence of the original HTTP or model request, determines what the client displays.

## Long-running media operations

Image and video generation often outlive the model round that requested them. Flip separates conversational planning from media execution:

1. The tool validates the request and available capability.
2. A pending artifact and durable media job are created.
3. The external or local provider performs the operation asynchronously.
4. Polling, callback, or worker state records the terminal result.
5. A unique delivery operation updates the artifact and conversation.
6. Where useful, a bounded model continuation composes a natural follow-up from the terminal facts.

Multi-step video chains carry chain and step identity. Two steps reaching terminal state together cannot start competing continuation turns over the same request.

## Error and retry behavior

| Condition | Behavior |
|---|---|
| **Unknown or malformed tool call** | The call is rejected or returned to the model with a typed validation error. |
| **Unauthorized effect** | The action is denied by product code and recorded as denied rather than described as executed. |
| **Transient backend failure** | The tool or owning job may retry within its bounded policy. |
| **Tool timeout** | The result records timeout state; the agent can continue with existing evidence or disclose the incomplete operation. |
| **Duplicate provider callback** | Durable job and artifact identities converge on one terminal update and one delivery path. |
| **Terminal media failure** | The artifact remains queryable with failed state and safe error facts; the conversation can receive an honest failure update. |
| **Citation validation failure** | Invalid or inaccessible citation references are repaired or removed before publication. |
| **Reader lacks source access** | Citation resolution denies the source while preserving the AI message and public execution metadata. |

## Selected private implementation paths

| Path | Responsibility |
|---|---|
| `lib/flip/synthesis/tools/` | Product tool schemas, dispatch, retrieval, document, media, and structured-state operations. |
| `lib/flip/synthesis/action_authority.ex` | Effect-level authorization for model-proposed operations. |
| `lib/flip/llm/arg_sanitizer.ex` | Safe argument handling for durable audit. |
| `lib/flip/llm/tool_call.ex` and `tool_call_audit.ex` | Tool-call state, normalized result, and audit projection. |
| `lib/flip/llm/source_ledger.ex` | Reader-facing citation and execution evidence. |
| `lib/flip/retrieval/` and `lib/flip/search/` | Internal retrieval and search services. |
| `lib/flip/pdf_session_cache.ex` | Bounded document-session state for PDF work. |
| `lib/flip/synthesis/artifact.ex` | Typed durable artifact model. |
| `lib/flip/synthesis/artifact_preflight.ex` | Artifact request validation and admission. |
| `lib/flip/synthesis/artifact_wire.ex` | Artifact references carried through model and client protocols. |
| `lib/flip/images.ex`, `lib/flip/video.ex`, and related directories | Durable image and video operations, provider jobs, chains, and delivery. |

[← Flip case study](../README.md)
