# Public Architecture Boundary

Flip is a production application with implementation details that stay out of public view: production prompts, credentials, vendor pricing, and internal configuration. Reviewers and public audiences still need to understand what the system does and how it is structured.

## Principle

This repository describes Flip as public architecture only. It documents capabilities and structure in conceptual terms, publishes diagrams and demo scenarios, and links only to the live public hosts `https://flip.engineering` and `https://flip.tech-demo.dev`.

## What is not published

Source code, source mirrors, production persona prompts, secrets, credentials, vendor pricing, internal URLs, and implementation-specific names are not published. The overview does not quote implementation source or expose exact internal configuration.

## Expected effect

Reviewers can evaluate the architecture without accessing implementation internals. The overview cannot answer implementation-level questions about prompts, exact algorithms, or configurations, and that is intentional.
