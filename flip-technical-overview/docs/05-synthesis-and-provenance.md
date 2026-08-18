# 05 — Synthesis and Provenance

## The essential distinction

Flip has two systems that both use models but have different authorship contracts:

1. **Conversation curation** transforms the structure around human-authored conversation.
2. **AI participation** creates new, explicitly AI-authored content and artifacts.

Treating both as generic “synthesis” obscures the product’s most important integrity guarantee.

## Conversation curation

<img src="../diagrams/synthesis-pipeline.svg" alt="Chat-to-forum synthesis flow" width="900" />

### Inputs

- eligible room and time range;
- messages visible to the curation workflow;
- reply/thread relationships;
- participant identities;
- pins/reactions or other signal;
- forum/community taxonomy;
- prior synthesis/linkback state;
- room-level configuration and feedback.

### Processing

A curation run can:

- identify coherent topics;
- exclude low-signal or ineligible material;
- choose or create a forum destination;
- order source messages;
- add short structural bridge context;
- merge with an existing topic where appropriate;
- preserve links to original messages;
- request additional evidence or a richer rendered artifact;
- produce a structured plan before applying product effects.

### Output

The durable output contains:

- forum thread/reply identities;
- copied or quoted source message material;
- source-message relationships;
- participant attribution;
- curation-run identity;
- bounded structural text;
- destination and tagging;
- linkback state;
- feedback/recuration state.

### Authorship invariant

Curation must not make an AI paraphrase appear to be a participant’s statement. Structural bridge text can explain ordering or context, but attributed claims remain tied to the source message.

That is stronger than “the prompt tells the model not to rewrite.” The durable schema retains the source identity needed to enforce and render the distinction.

## AI-authored replies

An AI participant uses product context and tools to write new content. Its output is:

- attributed to an explicit AI identity;
- linked to its trigger or parent message/thread;
- associated with citation/artifact records;
- constrained by the invoking actor/community;
- persisted as a product-native content type.

Unlike curation, rewriting is expected: the AI is composing its own answer. Integrity comes from visible authorship and evidence, not source-word preservation.

## Provenance layers

### 1. Source-message provenance

Answers:

- Which messages were selected?
- Who authored them?
- Where were they in the original conversation?
- Did the synthesized thread preserve reply order and participant identity?
- Can the reader navigate back?

### 2. Curation provenance

Answers:

- Which run produced the structure?
- Which configuration and forum destination applied?
- Was the result new, merged, linked back, or recurated?
- What participant feedback caused a later version?

### 3. Citation provenance

Answers:

- Which source was read?
- Which passage supports the AI claim?
- What citation identity appears in the final reply?
- Was the source external web, document, structured data, or internal product content?

### 4. Artifact provenance

Answers:

- Which tool and request produced the artifact?
- What inputs and source records did it use?
- Was it synchronous or asynchronous?
- Which terminal event and continuation attached it to the conversation?

### 5. Action provenance

Answers:

- Which user/event invoked the AI?
- Which AI participant acted?
- Which product authorization allowed the action?
- What durable receipt identifies the effect?

These records serve different readers and should not be collapsed into one opaque “agent trace.”

## Curation lifecycle

```text
crawl/select
  -> create synthesis run
  -> model-assisted topic plan
  -> validate source references and destination
  -> transactionally create/update forum structure
  -> enqueue linkback
  -> publish durable update
  -> collect participant feedback
  -> optional bounded recuration
  -> manual review when automation is exhausted
```

### Validation before apply

A curation plan can be rejected or repaired if it:

- references a source message outside the eligible set;
- loses participant identity;
- chooses an invalid forum destination;
- duplicates an already linked topic;
- produces structural text outside policy;
- creates malformed rich-data/render intents;
- violates feature or community configuration.

### Linkback

The source room should learn that a durable artifact exists. Linkback is an explicit asynchronous stage so a failure to post the link does not roll back the already committed forum artifact. The system can retry or repair linkback independently.

### Recuration

Participants can provide structured correction. Recuration is bounded and linked to the prior result. It does not silently overwrite provenance; later versions retain the causal path from feedback to changed structure.

## AI reply provenance lifecycle

```text
trigger
  -> origin scope
  -> model route
  -> tool calls
  -> source/citation/artifact records
  -> terminal draft
  -> output validation
  -> attributed reply
```

The reader-facing reply should not expose internal orchestration, but the product can retain enough evidence to debug:

- which provider/model route was selected;
- which tools ran;
- which source identities were minted;
- whether terminal recovery occurred;
- which durable artifacts were attached.

## Citations versus source links

A source-message link and a citation are not interchangeable:

- a **source-message link** establishes that a forum artifact derives from a particular conversation;
- a **citation** supports a claim made by an AI participant;
- an **internal content link** can be evidence in an AI answer;
- an **artifact dependency** establishes which data produced a chart or media result.

The UI can render them consistently while preserving their distinct semantics.

## Deduplication and merge

Durable knowledge quality depends on avoiding uncontrolled thread proliferation. The synthesis layer can compare candidate topics with existing forum state and choose:

- create new thread;
- append a reply;
- merge related source groups;
- skip already represented material;
- link to an existing thread;
- defer ambiguous placement.

The model may recommend a branch; code validates object identities and applies the effect transactionally.

## Reader trust model

A reader should be able to distinguish:

- human source text;
- AI-authored reply;
- AI-generated structural bridge;
- external citation;
- internal source link;
- generated artifact;
- later correction or recuration.

Flip’s provenance design is successful when this distinction is visible without requiring the reader to inspect an agent log.

## Privacy and deletion

Provenance introduces retention obligations. If a source message becomes inaccessible or is deleted under product policy, derived views must follow the defined deletion/redaction behavior. A public forum artifact must not retain private source text merely because it has a historical foreign key.

The architecture therefore requires policy for:

- visibility changes;
- message deletion/redaction;
- account deletion;
- artifact retention;
- citation cache retention;
- synchronization tombstones;
- audit access.

The precise policy is deployment-specific and intentionally not encoded as a universal public claim.

## Failure handling

| Failure | Result |
|---|---|
| Topic plan invalid | reject/repair before forum effect |
| Destination unavailable | select valid destination or defer |
| Source relationship missing | no unattributed copied text |
| Forum transaction fails | no phantom linkback |
| Linkback fails | preserve forum result; retry linkback |
| Feedback conflicts | retain explicit review state |
| Recuration limit reached | manual review rather than infinite model loop |
| Citation invalid | remove/repair unsupported AI claim |
| Artifact terminal failure | preserve failed artifact state and honest continuation |

## Why this matters

Many AI collaboration products retain only the generated summary. Flip retains the human conversation, the durable structure, and the relationship between them. That makes the output searchable without turning the model into an invisible co-author of the participants’ history.
