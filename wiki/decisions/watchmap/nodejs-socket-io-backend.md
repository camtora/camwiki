---
title: "Decision: Node.js + Socket.IO for real-time backend"
type: decision
tags: [decision, adr, nodejs, socket-io, real-time]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Node.js + Socket.IO for real-time backend

**Date:** 2026-04-17 (reconstructed from initial commit 5b2c94f)
**Status:** `accepted`

## Context

Watchmap needs to poll Tautulli every 2 seconds and push stream updates to all connected browsers in real-time. The backend is I/O-heavy (network polling) with no CPU-intensive computation.

## Options Considered

1. **Python (Flask/FastAPI)** — familiar for home server work, but async WebSocket support adds complexity; heavier container image.
2. **Go** — fast and efficient but adds build complexity; less appropriate for a small dashboard project.
3. **Node.js + Express + Socket.IO** — minimal boilerplate, native async I/O, Socket.IO provides bidirectional WebSocket with automatic HTTP long-polling fallback.

## Decision

Node.js 20 (Alpine) with Express and Socket.IO 4.7. Reasons:

- **Real-time fit:** Socket.IO's room model and event broadcasting align naturally with the "push update to all viewers when a stream changes" requirement.
- **Minimal container:** Node:20-alpine Dockerfile is 14 lines; no build step for the backend.
- **Language consistency:** Frontend is vanilla JS; same language reduces cognitive overhead.
- **I/O-heavy workload:** Node.js's event loop handles concurrent Tautulli polling, geolocation lookups, and WebSocket connections efficiently.

REST API endpoints were added later as a secondary interface for the tvOS client (which can't use Socket.IO without a third-party Swift dependency).

## Consequences

- CPU-intensive operations would block the event loop — not a concern for Watchmap's workload.
- Socket.IO adds ~1MB to the container image; acceptable at this scale.

## Related Pages

- [[wiki/projects/watchmap]]
- [[wiki/concepts/websocket-realtime-events]]

## Sources

- `raw/repos/watchmap` — backend/Dockerfile, backend/package.json, backend/server.js, initial commit 5b2c94f
