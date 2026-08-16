# 05 — Synthesis and Provenance

Synthesis output is not published without provenance: every AI-generated forum result is linked back to the discussion that produced it, and citations are verified before they reach readers.

<img src="../diagrams/synthesis-pipeline.svg" alt="Synthesis and provenance pipeline" width="760" />

## Source attribution

Forum threads and replies carry a source type — for example, chat, forum, or synthesis. A synthesis-created thread records the source discussion and source message that initiated it, so the generated outcome can be traced back through the exact conversation. Chat messages likewise retain a link to any synthesis thread they triggered.

## Citations

Web and PDF citations are AI-minted and quote-verified. Each citation is tied to a stable identity for the source, so a claim can be checked against the original material. Citation identity and citation quality are covered by dedicated verification paths.

## Reader-facing source ledger

The source ledger gives readers a view of the sources behind an AI message. Model activity is recorded separately for operational visibility. Together these mechanisms make provenance part of the output rather than an afterthought.

## Pipeline

A chat message triggers synthesis, the bounded agent gathers sources, the final response is submitted, and the resulting forum thread or reply is stored with its source attribution, citations, and ledger entries.
