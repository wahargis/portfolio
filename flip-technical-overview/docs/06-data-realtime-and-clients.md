# 06 — Data, Realtime, and Clients

Flip uses one authoritative server data store with two realtime paths: live web updates and a live-sync path for the native desktop client.

<img src="../diagrams/client-synchronization.svg" alt="Client synchronization" width="760" />

## Authoritative data

The server is the system of record for rooms, threads, messages, tags, and provenance. It owns writes and realtime fan-out; no client copy is treated as a competing database.

## Web realtime

The web client receives live updates through the server’s realtime layer. Chat rooms, forum threads, and their replies stay in sync as users interact.

## Native client sync

The native desktop client connects to the server API and uses a live sync service to keep its local view current. The client handles session refresh and update-required responses gracefully. Offline behavior is explicit: an offline indicator dims the UI, an in-memory optimistic send queue holds messages in order, and queued messages replay on reconnect. There is no durable offline mode today; local state is ephemeral and the authoritative copy is always the server.

## Client platforms

The native client targets desktop platforms, with mobile client work in progress. The same product experience is available on the web.
