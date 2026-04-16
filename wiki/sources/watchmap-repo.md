---
title: "Source: watchmap repo"
type: source
tags: [source, plex, tautulli, websocket, tvos, map]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: watchmap repo

**Raw file:** `raw/repos/watchmap`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

Watchmap is a real-time Plex stream dashboard: Node.js/Express + Socket.IO backend serving a Leaflet.js single-page app via Nginx. WebSocket events push session start/stop/progress in real time. Includes a native tvOS app (WatchMapTV, SwiftUI) with full visual parity to the web dashboard.

The key cross-repo detail: the backend writes geolocated session data to `camerontora.ca/app/data/locations.json`, which is bind-mounted read-only into the container and also consumed by the public-facing map on camerontora.ca.

## Key Facts / Data Points

- Backend: Node.js/Express + Socket.IO; geolocation cache 24h TTL via ip-api.com
- Frontend: single-page app (Leaflet.js + Socket.IO client) served by Nginx on port 5080
- WebSocket events: `event` (session), `metrics_update`, `stats_update`, `recent_added_update`
- tvOS app (WatchMapTV): SwiftUI, sidebar panels, footer stats, status banner (Phase 2)
- Image proxy endpoint `/img?...` for Tautulli/Plex posters and avatars
- Separate repo from docker-services; updated independently via `docker-compose build`

## Pages Updated

- [[wiki/projects/watchmap]] — created
