---
title: "Watchmap"
type: project
tags: [project, plex, tautulli, map, websocket, tvos]
created: 2026-04-16
updated: 2026-04-17
source_count: 2
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

**Historical markers:**
- Toggle shows past streaming locations (95 unique locations after deduplication)
- Owner locations: cyan gradient; other users: purple gradient; 14px dots behind active markers
- Deduplication by `label|isOwner` key (reduces ~382 raw → 95 unique)
- Data read from `camerontora.ca/app/data/locations.json` (same file used by public map)

**tvOS app (WatchMapTV):**
- Full visual parity with web dashboard
- Sidebar panels: Top Users and active Streams
- Footer stats: stream counts, transcode/direct breakdown, bandwidth, CPU, RAM
- Status banner with color-coded alerts (Phase 2)
- App icons included
- Polls REST API every 5 seconds (`/api/locations`, `/api/streams`, `/api/metrics`, `/api/top-users`); status every 60s

## tvOS Architecture Decisions

**WebKit not available on tvOS** — the initial plan was to use WKWebView to display the existing web dashboard. Apple restricts WebKit to internal use on tvOS. Solution: native SwiftUI app using MapKit — which provides a better TV experience with Siri Remote support.

**REST API instead of Socket.IO** — Socket.IO on Swift requires a third-party library. Instead, the backend exposes simple REST endpoints (`/api/locations`, `/api/streams`) that the tvOS app polls every 5 seconds. Slightly less real-time but no external dependencies.

**Deployment** — requires a paid Apple Developer Program ($99/year) for physical Apple TV deployment. Free/Personal Team accounts do not support tvOS device deployment.

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

## Key Decisions

- [[wiki/decisions/watchmap/tvos-native-swiftui|tvOS Native SwiftUI]] — WebKit unavailable on tvOS; native MapKit app with REST polling instead.
- [[wiki/decisions/watchmap/nodejs-socket-io-backend|Node.js + Socket.IO Backend]] — Real-time push to browsers; minimal container; I/O-heavy workload fits Node event loop.
- [[wiki/decisions/watchmap/single-file-frontend|Single-file Frontend]] — No build step; volume-mounted for instant reload; zero frontend dependencies.
- [[wiki/decisions/watchmap/ip-api-geolocation|ip-api.com Geolocation]] — Free, no API key; 24-hour TTL cache; LAN IPs mapped to configurable home location.
- [[wiki/decisions/watchmap/proc-mount-system-stats|Host /proc Mount for System Stats]] — Read-only /proc mount gives accurate real-time CPU/RAM/load; safer than Docker socket.

## Known Issues

**High**
- **No WebSocket reconnection strategy** — if the Socket.IO connection drops, the web client does not attempt to reconnect. The user must manually refresh the page to restore live updates.
- **`LOCATIONS_FILE` path hardcoded to camerontora.ca tree** — the backend hard-codes the output path to `camerontora.ca/app/data/locations.json`. If camerontora.ca is ever restructured or the bind mount path changes, writes silently fail with no error surfacing.

**Medium**
- **Multiple overlapping polling loops** — the backend runs separate setInterval loops for sessions, metrics, and geolocation with no coordination. Under high load or slow API responses, loops can overlap, causing duplicate Tautulli API calls.
- **Debug mode left in production** — pressing `D` on the web dashboard enables a debug overlay. This is not guarded by any auth check.
- **tvOS deployment requires paid Apple Developer Program ($99/year)** — the WatchMapTV app cannot be installed on a physical Apple TV without an active paid developer account. Personal/Free Team accounts cannot deploy to tvOS devices.

**Low**
- **3340-line monolithic HTML file** — the entire web frontend is a single `index.html` file. Adding features requires editing a file this large with no bundler or component isolation.
- **/proc mount fails silently** — if the host `/proc` is not mounted into the container, CPU/RAM stats return zeros or NaN without logging an error.

## Sources

- [[wiki/sources/watchmap-repo]] — full architecture, WebSocket events, tvOS app documentation
