# 02 — System Architecture

Flip is a server-backed web and desktop application with realtime collaboration, an asynchronous AI runtime, and a live sync path for native clients.

<img src="../diagrams/service-container-map.svg" alt="Flip service and container architecture diagram" width="760" />

## Core parts

- **Web and native clients** — the web client is the primary interactive surface; the native desktop client provides a persistent local product experience.
- **Server application** — owns chat, forums, accounts, permissions, and authoritative writes.
- **Data layer** — the system of record for rooms, threads, messages, tags, and provenance.
- **Realtime layer** — delivers live updates to web clients during chat and forum activity.
- **Agent runtime** — runs AI synthesis and reply work asynchronously, separate from latency-sensitive interaction.
- **Live sync service** — keeps the native client’s local view current with server changes while the server remains authoritative.
- **External source retrieval** — lets AI participants search and read public sources under governed tools.

## Separation of paths

The architecture keeps interactive paths, asynchronous AI paths, and client sync paths independent. That means live chat is not blocked by AI work, and AI runs can be bounded, retried, and audited without coupling them to realtime UI updates.

## Deployment shape

The production and demo environments are separate. Each runs its own data resources, credentials, and seed data; the demo is designed to be safely resettable. The public hosts are <https://flip.engineering> and <https://flip.tech-demo.dev>.
