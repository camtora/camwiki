---
title: "camerontora.ca"
type: project
tags: [project, website, nextjs, plex, map, oauth]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# camerontora.ca

Personal landing page and home dashboard at `camerontora.ca`. Features an interactive Plex watch history map and a service tile dashboard for authenticated users.

## Status

Current status: `active`
Last meaningful change: 2026-03-28 (Ombi/Overseerr → Seerr, SBCA added as flagship)

## Architecture

```
Internet → nginx-proxy (infrastructure) → Apache (443)
                                              → OAuth2 Proxy (4182)
                                                  → Next.js (3002)
```

Has its own OAuth2 Proxy instance separate from the shared infrastructure proxy. See [[wiki/infrastructure/oauth2-proxy]].

| Component | Port | Notes |
|-----------|------|-------|
| Apache | 443 | TLS termination for this stack |
| OAuth2 Proxy | 4182 | Google SSO, scoped to camerontora.ca |
| Next.js | 3002 | App server |

**Tech stack:** Next.js 15, React 19, Tailwind CSS 4, Leaflet, Docker Compose.

## Features

### Plex Watch History Map

Interactive world map showing geolocated Tautulli stream history.

- Cyan pins: owner streams
- Purple pins: other users' streams
- Green pulsing pins: live activity

**Data flow:**
1. Watchmap backend (or backfill script) fetches Tautulli history → geolocates via ip-api.com → writes to `app/data/locations.json`
2. Hourly cron (`0 * * * *`) runs `scripts/backfill-locations.ts` to keep history current
3. `/api/tautulli` reads `locations.json` + fetches live activity from Tautulli
4. `MapSection.tsx` renders markers

**Cross-repo dependency:** `app/data/locations.json` is written by the [[wiki/projects/watchmap]] backend and bind-mounted read-only into its container. The file is gitignored for privacy.

### Service Tiles

Visible to authenticated users. Organized by category:

| Category | Services |
|----------|---------|
| Flagship apps | SBCA, Who's Up |
| Personal projects | Haymaker, Rotosync |
| Backend services | Radarr, Sonarr, Jackett, Tautulli, Netdata, etc. |

### Status Banner

Pulls outage severity from the status API and displays a banner (major/minor/degraded/healthy). Source: `app/api/status/route.ts`.

### Auth Flow

1. User clicks "I'm Cam!" → redirects to `/oauth2/start`
2. OAuth2 Proxy handles Google SSO
3. Success sets `x-forwarded-email` header
4. `page.tsx` checks email against `allowedEmails` array for authorization
5. Authorized users see private service tiles

### Contact + Access Request Forms

Both post to Discord webhook. `app/api/contact/route.ts` (contact), `app/api/plex-access-request/route.ts` (Plex library access request).

## Infrastructure

- Reverse proxy: [[wiki/infrastructure/nginx-reverse-proxy]] routes `camerontora.ca` to port 3002
- Docker network: `camerontoraca_default` (172.20.0.0/16)
- Auth: [[wiki/infrastructure/oauth2-proxy]] (own instance at port 4182)
- Monitoring: [[wiki/infrastructure/gcp-external-monitoring]] checks `camerontora.ca` every 5 min
- DNS failover: `camerontora.ca` is one of the three records that fail over to GCP — see [[wiki/concepts/dns-failover]]

## Integrations

- [[wiki/projects/plex-media-server]] — Tautulli API for map data and library stats
- [[wiki/projects/watchmap]] — writes `data/locations.json` consumed by the map
- [[wiki/infrastructure/status-dashboard]] — status API consumed by the status banner

## Key Files

| File | Purpose |
|------|---------|
| `app/app/page.tsx` | Main page, auth header check, layout |
| `app/app/lib/subdomains.ts` | Service tile definitions (public/private, categories) |
| `app/app/components/MapSection.tsx` | Leaflet map component |
| `app/app/api/tautulli/route.ts` | Map data API |
| `app/data/locations.json` | Geolocated watch history (gitignored, written by Watchmap) |
| `app/scripts/backfill-locations.ts` | Historical IP geolocation backfill |

## Change Log

- 2026-03-28: Replaced Ombi/Overseerr tiles with Seerr; added SBCA as flagship app
- Earlier: Added Who's Up as flagship app with access request flow
- Earlier: Added 3D dancing avatar, status banner, service tiles redesign

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/camerontora-ca-repo]] — full app architecture, map data flow, auth flow, service tiles
