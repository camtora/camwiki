---
title: "Watchmap"
type: project
tags: [project, plex, tautulli, map, websocket, tvos]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Watchmap

Real-time geospatial dashboard for visualizing active Plex streaming sessions. Shows live viewer locations on an interactive world map with detailed streaming metrics. Also has a native tvOS app.

## Status

Current status: `active`
Last meaningful change: 2026-04-16 (status banner + tvOS Phase 2 complete)

## Architecture

```
┌─────────────────────────────────────────┐
│  watchmap-backend (Node.js/Express)     │
│  + Socket.IO                            │
│      │                                  │
│      ├── Tautulli API (polling)         │
│      ├── ip-api.com (geolocation)       │
│      └── writes → camerontora.ca/       │
│            app/data/locations.json      │
├─────────────────────────────────────────┤
│  watchmap-web (Nginx, port 5080)        │
│  Single-page app: Leaflet.js + Socket.IO│
└─────────────────────────────────────────┘
```

## Features

**Web dashboard:**
- Interactive Leaflet map (smart card layout, no overlapping markers)
- WebSocket real-time updates (start/stop/progress events)
- Stream detail panel — codec info, transcode status, playback metrics
- Streams sidebar — toggle active/recent streams; click to open detail panel
- Metrics bar — active streams, transcodes, direct play counts, bandwidth
- Top Users sidebar — top 50 by watch hours (7d/30d/1y toggle)
- Recently Added — latest Plex library additions
- Status banner showing home server health

**tvOS app (WatchMapTV):**
- Full visual parity with web dashboard
- Sidebar panels: Top Users and active Streams
- Footer stats bar
- Status banner (Phase 2)
- App icons included

## Cross-Repo Dependency

Watchmap backend writes geolocation data to `camerontora.ca/app/data/locations.json`. This file is bind-mounted read-only into the Watchmap container and read by [[wiki/projects/camerontora-ca]]'s map API. Historical location data for the public-facing map lives here — it is gitignored for privacy.

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `TAUTULLI_URL` | — | Tautulli instance URL |
| `TAUTULLI_KEY` | — | Tautulli API key |
| `PLEX_URL` | — | Plex server URL (optional) |
| `PLEX_TOKEN` | — | Plex token (optional) |
| `POLL_MS` | 2000 | Session polling interval |
| `HOME_LAT/LON` | — | LAN viewer fallback location |

Geolocation results cached for 24 hours (ip-api.com TTL).

## Infrastructure

- URL: `watchmap.camerontora.ca` (OAuth2 protected, port 5080)
- Deployed in [[wiki/projects/docker-services]] Compose stack (built from local Dockerfile)
- Reverse proxy: [[wiki/infrastructure/nginx-reverse-proxy]]
- Update separately from other services: `docker-compose build watchmap-backend && docker-compose up -d watchmap-backend watchmap-web`

## Integrations

- [[wiki/projects/plex-media-server]] — Tautulli API for stream data and Plex for library/poster images
- [[wiki/projects/camerontora-ca]] — writes `locations.json` consumed by the public map

## Sources

- [[wiki/sources/watchmap-repo]] — full architecture, WebSocket events, tvOS app documentation
