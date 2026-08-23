# Flip Clients and Deployment

Flip serves the same community and AI product through a Phoenix web application and React-based desktop and mobile clients. The clients differ in rendering, local storage, operating-system integration, and synchronization mechanics, but they do not define separate domain rules or authorization models.

<table>
<tr>
<td width="50%"><img src="../assets/flip.engineering.png" alt="Flip product interface" width="100%" /></td>
<td width="50%"><img src="../assets/flip.tech-demo.dev.png" alt="Flip synthetic technical environment" width="100%" /></td>
</tr>
</table>

## Shared application semantics

Commands from LiveView, API routes, Channels, and native clients enter the same domain modules. A client does not receive weaker authorization or a second interpretation of message, forum, synthesis, AI, or artifact lifecycle because it uses a different transport.

The server remains authoritative for:

- actor identity and membership;
- read and effect authorization;
- message and forum creation;
- synthesis and AI workflow state;
- citations and artifacts;
- feature and deployment capability;
- accepted mutation identity and final transaction state.

Clients can render optimistically, but an optimistic state is not a committed product effect.

## Web client

The primary web application uses Phoenix LiveView for server-driven interactive views and Phoenix Channels for realtime topics.

LiveView is suited to product operations whose state already lives in the Phoenix application: account flows, room and forum views, administration, workflow status, and rich server-rendered interaction. Commands call the same contexts used by background jobs and API endpoints.

Channels carry immediate room, user, presence, typing, notification, and workflow events. Durable events are published after the corresponding transaction commits. Ephemeral signals are not treated as a substitute for database state.

## Native and installable clients

The React and TypeScript client targets desktop, mobile, and installable web use. Tauri provides desktop packaging and operating-system integration; Capacitor provides mobile packaging; the PWA surface uses the same product components where platform capability permits.

Native clients add concerns not present in a server-rendered browser session:

- local persistent storage and cache migration;
- optimistic mutation state;
- offline or interrupted synchronization;
- push notifications;
- deep links and protocol handling;
- file and media selection;
- native release and update compatibility;
- authentication handoff and credential storage;
- background and resumed application state.

These capabilities are negotiated rather than assumed. A deployment or client version can advertise which authentication, sync, push, upload, reaction, emoji, deep-link, and AI operations it supports.

## Durable synchronization

Electric projects authorized durable tables to the React client through shape subscriptions. The client maintains a local view that can be queried and rendered without requesting every object independently.

### Optimistic mutation

A client mutation receives a stable identity and updates local state immediately where appropriate. The command is sent to the Phoenix application, which validates the actor and commits the transaction. Subsequent synchronized changes confirm, replace, or reject the optimistic state.

The transaction identity allows the client to distinguish its own committed mutation from an unrelated server update and avoids showing the same message or reaction twice when the authoritative row arrives.

### Reconnect and resubscription

After a disconnect, the client resumes durable synchronization from the server-backed stream and rebuilds the current view. It does not need a replay of every typing indicator, presence transition, or transient socket notification that occurred while offline.

### Conflict and rejection

The client can create a tentative state that the server rejects because membership, authorization, moderation, feature configuration, or target state changed. Reconciliation removes or marks the tentative mutation and surfaces the server result rather than leaving the local copy as an alternate truth.

## Ephemeral realtime state

Presence, typing, and other short-lived coordination signals remain on Channels and PubSub. Their value depends on current liveness, not historical replay.

This separation keeps durable sync focused on product objects while allowing the realtime layer to deliver high-frequency state that can expire harmlessly.

## AI and artifact delivery to clients

An AI turn can cross several visible states:

1. admitted or queued;
2. generating or using tools;
3. final reply posted;
4. artifact pending;
5. artifact completed or failed;
6. continuation or message update delivered.

Those states are represented by durable activity, message, and artifact records where reconnection matters. Ephemeral token or phase streaming can improve the live experience but is not required to reconstruct the final conversation.

Clients render typed artifacts according to their lifecycle and media metadata. A pending video is not rendered as a broken completed file; a failed operation retains a terminal status; a completed artifact can expose preview, playback, editing, or download actions supported by the current client.

## Deployment components

A typical deployment contains:

- Phoenix application processes and the HTTP endpoint;
- PostgreSQL as product authority;
- Oban queues for AI, synthesis, media, notification, cleanup, and other durable work;
- PubSub and Channels for realtime delivery;
- Electric synchronization for native durable state;
- object or file storage for uploads and generated media;
- configured model, search, document, and media providers;
- telemetry, logging, health, and operational controls;
- web, desktop, mobile, and PWA client builds appropriate to the deployment.

Optional capability is explicit. A deployment without one provider or native integration can continue to serve ordinary community features and can suppress unavailable actions through capability metadata.

## Production and synthetic environments

Flip maintains a separation between the product environment and the public synthetic technical environment.

### Product environment

The product environment contains real accounts, communities, messages, operational configuration, credentials, moderation authority, and deployment state. Those data and controls are not mirrored into the public portfolio.

### Synthetic technical environment

The synthetic environment demonstrates product surfaces and architecture using generated accounts, communities, messages, documents, media, workflows, and failure cases. It can expose representative state and instrumentation without granting access to production content or administration.

The environments can share application code and public interface contracts while using separate databases, credentials, storage, provider configuration, and authority roots.

## Provider boundary

Inference and media providers are external capabilities behind product-owned adapters. A provider receives the bounded request and tool protocol required by the admitted operation; it does not become the owner of community identity, room access, workflow state, or publication.

Local and hosted routes can coexist. Route selection considers capability, context, latency, cost, privacy, and health, while the final product state remains committed through Flip.

## Selected private implementation paths

| Path | Responsibility |
|---|---|
| `flip-client/` | React and TypeScript desktop, mobile, and PWA product client. |
| `lib/flip/sync.ex` and sync controllers | Durable shape subscription, mutation identity, and client reconciliation. |
| `lib/flip_web/channels/` | Realtime room, user, presence, and event delivery. |
| `lib/flip_web/live/` | Server-rendered web product surfaces. |
| `lib/flip/capabilities.ex` | Server capability document consumed by clients. |
| `lib/flip/client_releases.ex` | Native release and compatibility state. |
| `lib/flip/client_telemetry.ex` | Client operational telemetry. |
| `lib/flip/application.ex` | Supervised runtime composition. |
| `config/runtime.exs` | Deployment-time feature, provider, storage, queue, and endpoint configuration. |

[← Flip case study](../README.md)
