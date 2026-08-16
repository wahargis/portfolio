# 01 — Product and Problem

Flip addresses a coordination problem: teams need fast, low-friction conversation and a durable record of what was decided. Chat alone loses decisions in the stream; forums alone are too slow for live exchange. Flip combines both.

The web product surface is a Phoenix LiveView application with routes for `/chat`, `/server/:server_id/chat`, `/forum`, and `/server/:server_id/forum`. Real-time delivery runs over Phoenix channels (`RoomChannel`, `ThreadChannel`, `ForumChannel`). The chat and forum domains are `Flip.Chat` and `Flip.Forum`, backed by PostgreSQL migrations for rooms, threads, and tags.

Configurable AI participants are a central capability. Rooms carry a `synthesis_config` map with fields such as `enabled`, `model`, `max_tool_rounds`, and `ai_bridging`, plus an `ai_briefing`. Server-level settings constrain AI reply concurrency. Personas are versioned through `Flip.Synthesis.Persona`. This means a room can be configured to have AI summarize, bridge, or curate within explicit bounds — not as an open-ended chatbot, but as a governed participant with a specific brief.

The product also maintains provenance. Forum threads and replies carry a `source` enum (`chat`, `forum`, or `synthesis`), and synthesis-created threads must record the source channel and message. This connects AI output back to the discussion that produced it.

The client surface spans a LiveView web application and a Tauri desktop client. The web app is feature-gated for chat, forum, and synthesis. The desktop client is a separate Tauri 2.x application with desktop builds published as release artifacts and production defaults pointing at `https://flip.engineering`.

The problem this solves is not just "chat with AI" but "discuss, decide, and verify in one system."
