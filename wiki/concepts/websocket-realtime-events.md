---
title: "WebSocket / Real-time Events"
type: concept
tags: [concept, websockets, socket-io, realtime, events]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# WebSocket / Real-time Events

WebSockets provide a persistent, bidirectional TCP connection between client and server. Unlike HTTP (request → response), either side can send messages at any time. Socket.io is a library built on WebSockets that adds rooms, namespaces, automatic reconnection, and fallback to HTTP long-polling.

## How It Is Used Here

| Project | Library | Use case |
|---------|---------|----------|
| [[wiki/projects/whosup\|Whosup]] | Socket.io 4.8 | Join requests, approvals, group chat, presence updates |
| [[wiki/projects/watchmap\|Watchmap]] | Socket.IO | Real-time Plex stream location push to map |

## Key Properties / Mechanics

### Socket.io Rooms

Rooms are named channels within a server. A client joins a room; the server broadcasts to a room. In Whosup, each activity automatically gets a chat `Room` — joining a room grants access to both group chat messages and presence update events.

```
Client → join-room(roomId)
Server → broadcasts new-message, member-joined, presence-updated to all room members
```

### Event Patterns in Whosup

| Direction | Event | Payload | Trigger |
|-----------|-------|---------|---------|
| Client → Server | `join-room` | `{ roomId }` | User opens activity |
| Client → Server | `send-message` | `{ roomId, content }` | User sends chat message |
| Server → Client | `new-request` | request object | Sent to host when someone requests to join |
| Server → Client | `request-approved` | `{ presenceId }` | Sent to requester on approval |
| Server → Client | `request-denied` | `{ presenceId }` | Sent to requester on denial |
| Server → Client | `presence-updated` | presence object | Sent to room on any change |
| Server → Client | `presence-ended` | `{ presenceId }` | Sent to room when activity ends |

### Authentication over WebSocket

WebSocket handshakes go through the same Nginx + Express auth middleware as HTTP requests. JWT tokens are validated on the initial connection upgrade. Whosup's auth middleware runs before the Socket.io connection is established.

### Rate Limiting

WebSocket connections carry the same abuse risks as HTTP. Whosup enforces:
- **10 messages/second per IP** — prevents chat flooding
- **20 connections per IP** — prevents connection exhaustion

### Polling as a Fallback

Clarity's SMS verification screen polls `GET /verification/sms/status` every 2 seconds via a `LaunchedEffect` loop in `SmsVerifyScreen.kt`. This is explicit HTTP polling, not WebSocket — a deliberate simplicity trade-off. A WebSocket push would be more elegant but adds infrastructure for a human-speed interaction (30–120 second wait). See [[wiki/decisions/clarity/twilio-sms-verification]].

### PM2 Cluster and Sticky Sessions

Whosup runs under PM2 in cluster mode (`instances: 'max'`). Socket.io requires **sticky sessions** in a cluster — a client must always hit the same worker, because Socket.io room state is in-memory per worker. Without sticky sessions, events broadcast to a room on worker A won't reach clients connected to worker B. Whosup's nginx config must be verified to handle this.

## Relationships to Other Concepts

- [[wiki/concepts/jwt-authentication]] — WebSocket connections authenticate via JWT on the handshake
- [[wiki/concepts/docker-compose-networking]] — Socket.io server runs inside a Docker container; Nginx proxies WebSocket upgrades

## External References

- Socket.io docs: https://socket.io/docs/v4/
