# 05 — Synthesis and Provenance

Synthesis output is not published without provenance. Two provenance systems work together: source attribution on forum/chat records, and AI-minted citations with a source ledger.

Forum threads and replies carry a `source` enum with values `chat`, `forum`, and `synthesis`. A synthesis-created thread must record `source_channel_id` and `source_message_id`, linking the thread to the exact discussion that produced it. Chat messages store `synthesis_thread_id`, `synthesis_run_id`, and `synthesis_position`, so a generated thread can be traced back through the message that initiated it.

Web and PDF citations are AI-minted and quote-verified. `Flip.Media.WebCitation` stores a web citation identity hash so a citation is tied to a stable identity, and `Flip.Media.PdfCitation` handles PDF sources. Citation identity and citation quads are covered by dedicated tests in the server suite.

The reader-facing source ledger is `Flip.LLM.SourceLedger`, exposed through a `message_source_ledger` API controller. This gives readers a view of the sources behind an AI message. LLM call audit tables and `Flip.LLM.CallAudit` record model activity separately for operations.

The synthesis pipeline ties these together: a chat message triggers synthesis, the bounded tool loop gathers sources, the terminal plan is submitted, and the resulting forum thread or reply is stored with its source fields, citations, and ledger entries. Attribution and citation-attribution tests in the server suite cover these paths.

The key property is that citations are minted by the model but verified before acceptance, so provenance is part of the output rather than an afterthought added to free-form text.

<img src="../diagrams/synthesis-pipeline.svg" alt="Synthesis and provenance pipeline" width="760" />
