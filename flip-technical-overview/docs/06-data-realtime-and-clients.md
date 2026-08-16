# 06 — Data, Realtime, and Clients

Flip's data layer is PostgreSQL 16 with `wal_level=logical`, deployed as the `db` service in `docker-compose.yml`. The application uses Ecto and Postgrex, with migrations owned by the Phoenix application. ElectricSQL (`electricsql/electric:1.4.6`) consumes the logical WAL and exposes shape subscriptions to clients.

Realtime has two paths. Web users get Phoenix LiveView and Phoenix channels (`RoomChannel`, `ThreadChannel`, `ForumChannel`). Native/desktop users get a Tauri 2.x client that connects to the Phoenix API and to ElectricSQL.

The desktop client is built with Tauri, Rust, React, and TypeScript. Its Electric client implements in-memory `ShapeStream`/`Shape` subscriptions with ref-counted dedup and handles 401 mid-session refresh and 426 update-required responses. Offline behavior is explicit: an offline indicator dims the UI, and an in-memory optimistic send queue holds messages FIFO and replays on reconnect. There is no durable offline mode today; shape rows stay in memory and the authoritative copy is always the server database.

Desktop builds are published for macOS aarch64 and x86_64, Windows x64 (MSI and NSIS), and Linux x86_64 (AppImage and deb). The Tauri updater points at a versioned updater-manifest endpoint on the production host. Mobile (iOS/Android) build pipelines are in active development.

The split is deliberate: Phoenix owns authoritative writes and realtime fan-out for web; ElectricSQL owns live read-path sync to the desktop client; the client's local state is ephemeral rather than a competing database.

<img src="../diagrams/client-synchronization.svg" alt="Client synchronization" width="760" />
