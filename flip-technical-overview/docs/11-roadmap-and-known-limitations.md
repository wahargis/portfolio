# 11 — Roadmap and Known Limitations

This page records current product limitations and near-term directions. It is a snapshot of the public architecture, not a commitment.

## Desktop sync is live, not a local database

The native client keeps its synced view in memory and replays queued sends on reconnect. It is not a durable offline mode; a persistent local replica is a future consideration. Until then, the authoritative copy is always the server.

## Mobile clients are in progress

Mobile build work is underway, while the current public native surface is desktop. Desktop releases remain the primary native client channel.

## The demo is synthetic and capped

The technical demo is seeded with versioned synthetic data, separate credentials, and capped model/tool usage. It is designed for reviewer scenarios, not as a production-equivalent environment.

## AI participants are bounded by design

Round caps, deadlines, token envelopes, and tool budgets are intentional limits. They keep runs predictable and safe, and they mean some deeply exploratory tasks are better split across multiple synthesis passes.

## Near-term work

Current directions include: publishing mobile client artifacts, evaluating a durable local cache for the native client, expanding the demo seed set with additional synthetic scenarios, and continuing to harden retrieval and citation quality.
