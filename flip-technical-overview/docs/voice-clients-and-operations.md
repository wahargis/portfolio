# Voice, clients, and production operation

Flip supports web, PWA, desktop, and mobile clients. Durable application data and live media use different transport paths because they have different latency and delivery requirements.

## Client surfaces

The shared React and TypeScript interface is packaged for:

- Browser and PWA use.
- Tauri desktop applications.
- Capacitor mobile applications.
- Responsive layouts for chat, forums, Library, Studio, and room controls.

The clients keep local read state for fast navigation and offline-tolerant views. Server-side checks still control membership, media grants, provider work, and durable state transitions.

## Durable synchronization

PostgreSQL is the source for messages, posts, replies, jobs, library items, and room records. Electric synchronization projects selected tables and changes to clients. The client can render local data immediately and receive later server changes without treating its local cache as the final authority.

Phoenix Channels carry transient events that do not need the same persistence model, including typing, presence, progress, and live-room control messages.

## Voice and video join flow

A participant requests access to a room through the application server. The server checks the user, room, membership, event state, and requested media role. It then issues a signed, short-lived join grant.

The media client presents that grant to the self-hosted SFU. Phoenix coordinates application room state and signaling. The SFU handles audio, camera, and screen-share packets.

Private signaling messages are addressed to the intended participant. They are not sent as a general room broadcast.

## Room operation

The live-room surface includes:

- Participant and presence state.
- Audio mute controls.
- Camera and screen-share state.
- Scheduled event relationships.
- Join and leave handling.
- Server-authorized role changes.
- Selection of active audio streams for client delivery.
- Recovery when a participant reconnects.

Durable room and event records remain in PostgreSQL. Ephemeral media and connection state remain in the live media path.

## Service supervision

The Phoenix application starts database, PubSub, endpoint, job, AI, provider-health, media, and reconciliation services under supervision. A failed worker can restart without restarting the complete product. Background schedulers repair incomplete Personal AI and media state.

Integration settings and encrypted credentials are loaded on the server. Provider probes and circuit breakers stop known failing routes from receiving unrestricted new work.

## Public and production environments

The production site and public technical demo use separate environment configuration. The public portfolio does not include production credentials, private messages, provider secrets, private network paths, or security-sensitive rate and abuse thresholds.
