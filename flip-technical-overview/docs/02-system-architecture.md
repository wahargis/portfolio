# 02 — System Architecture

Flip is an Elixir/Phoenix application with PostgreSQL, Oban, and ElectricSQL as core infrastructure.

The Phoenix stack uses Phoenix 1.8.9 and Phoenix LiveView. The web layer exposes LiveView pages for chat and forum surfaces plus REST/JSON API routes. Real-time messaging is carried over Phoenix channels for rooms, threads, and forums. A router pipeline enforces feature gates for `:chat`, `:forum`, and `:synthesis`, so capabilities can be enabled or disabled per deployment.

The domain is organized around Elixir contexts. `Flip.Chat` owns rooms, messages, and channel behavior; `Flip.Forum` owns threads, replies, tags, and source attribution. AI work is separate: `Flip.Synthesis` contains the tool loop, tools, dispatch, and reply worker. Supporting contexts cover search, retrieval, LLM access, media, and audit.

Background work runs through Oban 2.19. The application supervision tree starts Oban with feature-gated queues and crontab, the Phoenix endpoint, Finch for HTTP, and several supervised caches and workers. AI reply jobs and tool dispatch run under their own task supervisors.

The deployment unit is an Elixir release built by a Dockerfile, with a non-root runtime user and healthcheck. `docker-compose.yml` defines three services: `db` (PostgreSQL 16 with `wal_level=logical`), `electric` (ElectricSQL 1.4.6), and `app` (Phoenix on port 4001). Persistent volumes hold PostgreSQL data, uploads, and media recovery data.

The native client is a Tauri 2.x desktop application. It connects to the Phoenix API and to ElectricSQL for live sync, with production defaults pointing at `https://flip.engineering`.

The architecture separates interactive paths (LiveView/channels), asynchronous AI paths (Oban + tool loop), and sync paths (Electric shapes), which keeps latency-sensitive interaction independent of bounded AI work.

<img src="../diagrams/service-container-map.svg" alt="Service and container architecture" width="760" />
