# Technical Demo Scenarios

Reviewer path: open <https://flip.tech-demo.dev>, sign in with a stable demo account, and work through the three scenarios below. The demo is a separate environment profile with synthetic data and capped credentials; it is not production.

## Production vs demo separation

| Concern | Production | Technical demo |
|---|---|---|
| Host | `flip.engineering` | `flip.tech-demo.dev` |
| Data | Authorized production | Versioned synthetic seed data |
| Database/storage | Production resources | Separate resources |
| Credentials | Production routing | Separate capped credentials |
| Accounts | Real users | Stable demo accounts |
| Reset | Normal retention | Reproducible reset |
| Client target | Default production | Explicit demo profile |
| UI | Canonical product | Persistent technical-demo banner |

## Environment rules

1. **Synthetic seed data only.** The demo contains versioned synthetic seed data. No production data is copied, sampled, or derived into the demo.
2. **Reproducible reset.** The demo is resettable from its versioned seed set on demand. A reset returns all rooms, threads, messages, citations, and accounts to the seeded state.
3. **Stable demo accounts.** Reviewers use stable demo accounts created from the seed set. Demo accounts are not real users and do not work against production.
4. **Capped credentials.** The demo uses separate credentials with capped limits. Production provider keys and production database credentials are never referenced by demo configuration.
5. **Persistent banner.** The demo UI shows a persistent technical-demo banner so reviewers always know they are not in production.
6. **Explicit demo profile.** Web and desktop clients select the demo environment explicitly; production remains the default.

## Scenario 1: Retrieval + external search + structured tool + citations

1. Open the demo host and sign in with a stable demo account.
2. Start a chat in a seeded room whose AI participant has synthesis enabled.
3. Ask a question that requires current external information (for example, a question about a public documented event).
4. The AI participant should run source discovery and Brave Search, pass results through the retrieval cascade, and use a read tool (for example, `read_webpage`) as a structured tool call.
5. Confirm the response includes AI-minted, quote-verified citations and that the source ledger shows the sources behind the message.

Expected result: the reply cites external sources with verifiable quotes, and no production data appears.

## Scenario 2: Chat-to-forum synthesis with source-message provenance

1. In a seeded room, hold a short chat exchange that reaches a decision or plan.
2. Trigger synthesis from the chat.
3. Confirm a forum thread is created with `source = synthesis`.
4. Confirm the new thread records the source channel and source message that initiated the synthesis.
5. Open the source ledger for the synthesis result and confirm the lineage is visible.

Expected result: the forum thread is durably linked back to the exact chat message that produced it.

## Scenario 3: Prior decision + later evidence retrieval and relationship explanation

1. Use the seeded prior decision stored as a forum thread (created by an earlier synthesis).
2. Ask the AI participant to re-examine that decision against a later public source.
3. Confirm the system retrieves both the prior decision and the later evidence.
4. Confirm the response explains the relationship between the prior decision and the later evidence, with citations for the new source and a link back to the prior thread.

Expected result: the system demonstrates retrieval over both internal provenance and external sources, and explains how the new evidence relates to the old decision.
