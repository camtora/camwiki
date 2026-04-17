---
title: "Who's Up"
type: project
tags: [project, social, ios, swift, nodejs, postgis, maps]
created: 2026-04-16
updated: 2026-04-16
source_count: 2
---

# Who's Up

Map-first social app for discovering and joining nearby activities. Users broadcast that they're available ("up for surfing, coffee, hiking") and discover others doing the same within a geographic area. Hosted at `whosup.camerontora.ca`.

## Status

Current status: `active`
Last meaningful change: 2026-03-31 (reverted login tagline)

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         iOS App (SwiftUI)                        │
│  Map Screen  │  Host Screen  │  Activities  │  Connections       │
│                           API Client + Services                  │
└───────────────────────────┬─────────────────────────────────────┘
                            │ HTTPS + WebSocket
                            ▼
                     ┌─────────────┐
                     │ Nginx Proxy │  (SSL, rate limiting)
                     └──────┬──────┘
                            ▼
              ┌──────────────────────────────┐
              │  Node.js/Express (PM2 cluster)│
              │  Routes: /auth /presence /map │
              │          /activities /rooms   │
              │  Auth Middleware + Socket.io  │
              │  Prisma ORM                  │
              └──────────────┬───────────────┘
                             ▼
                  ┌──────────────────────┐
                  │ PostgreSQL + PostGIS  │
                  └──────────────────────┘
```

| Component | Tech |
|-----------|------|
| Backend | Node.js 20, Express, TypeScript |
| Database | PostgreSQL 15+ with PostGIS (geospatial queries) |
| ORM | Prisma 5.22 |
| Real-time | Socket.io 4.8 (WebSocket events) |
| Auth | JWT (7-day lifetime, stored in iOS Keychain) + Apple Sign In + Google Sign In |
| Validation | Zod |
| iOS app | SwiftUI, iOS 17+, Xcode 15+ |
| iOS auth | Sign in with Apple |
| iOS maps | MapKit + Core Location |
| Deployment | Docker Compose + PM2 cluster mode |

## Database Schema

Key entities and their critical columns:

**Area** — geographic regions where the app is active.

| Column | Description |
|--------|-------------|
| `id` | String primary key (e.g., `area_muskoka`) |
| `locationFuzzingLevel` | Enum: `LOW` (~200m), `MEDIUM` (~500m), `HIGH` (~1km) |
| `defaultRadiusKm` | Service radius in km |
| `centerLat/Lng` | Region center point |

**Presence** — an active "I'm Up" session (core entity).

| Column | Description |
|--------|-------------|
| `latCoarse / lngCoarse` | **Fuzzed** coordinates — shown to everyone on map |
| `latExact / lngExact` | **Exact** coordinates — revealed only to approved members |
| `activity` | Activity type string |
| `expiresAt` | When presence leaves the map |
| `maxParticipants` | Max allowed; full activities filtered from map view |
| `roomId` | Auto-created chat room |

**ActivityRequest** — join request state machine.

```
PENDING → APPROVED  (requester joins room, gets exact location)
PENDING → DENIED    (requester sees denial)
```

**Other entities:** `User` (photo[], presences[], likes, blocks, reports), `Room` → `RoomMember` + `RoomMessage`, `Thread` (mutual likes) → `Message`, `Block`, `Report`.

Key indexes: `Presence(areaId, expiresAt)` for map queries; `ActivityRequest(presenceId, status)` for pending requests.

## Privacy Architecture

Three-layer system protecting exact user locations:

1. **Location fuzzing by area** — `latCoarse`/`lngCoarse` are stored fuzzed at creation based on area's `locationFuzzingLevel` (LOW ≈ 200m, MEDIUM ≈ 500m, HIGH ≈ 1km).
2. **Coarse vs exact** — map shows only `latCoarse`/`lngCoarse`; `latExact`/`lngExact` revealed only after host approves join request.
3. **Distance buckets** — distances shown as `"<1km"`, `"1–5km"`, `"5–10km"`, `"10–20km"`, `"20km+"` — never exact km.

Map view additionally filters: blocked users, users who blocked requester, activities requester was denied from, full activities (at max participants).

## Real-time Events (Socket.io)

| Direction | Event | Trigger |
|-----------|-------|---------|
| Client → Server | `join-room`, `leave-room`, `send-message` | Group chat actions |
| Server → Client | `new-message`, `member-joined`, `member-left` | Room updates |
| Server → Client | `new-request` | Sent to host on join request |
| Server → Client | `request-approved`, `request-denied` | Sent to requester |
| Server → Client | `presence-updated`, `presence-ended` | Sent to room members |

WebSocket rate limit: 10 messages/sec per IP, connection limit: 20 per IP.

## Activity Types

| Activity | Icon | Color |
|----------|------|-------|
| Boating | sailboat.fill | Blue |
| Golf | figure.golf | Green |
| Patio Drinks | wineglass.fill | Purple |
| Surf | figure.surfing | Cyan |
| Yoga | figure.yoga | Pink |
| Wellness | heart.fill | Red |
| Hiking | figure.hiking | Orange |
| Sauna/Cold Plunge | thermometer.sun | Yellow |
| Beach | beach.umbrella.fill | Yellow |
| Dinner | fork.knife | Brown |
| Coffee | cup.and.saucer.fill | Brown |
| Exploring | binoculars.fill | Teal |

Activities can be filtered by region relevance (e.g., Surf for beach areas, Boating for lake areas).

## Features

- **Activity broadcasting** — post what you're "up" for with timing, location, category, and details
- **Map discovery** — see nearby active activities on a map
- **Join requests** — request to join with an optional message; host approves or denies
- **Group chat** — automatic room per activity, opened on approval
- **Privacy controls** — location fuzzing until approved; age ranges; distance buckets
- **Connections** — network of people you've done activities with
- **Direct messaging** — chat with connections

## Launch Regions

| Region | Location | Privacy radius | Geo radius |
|--------|----------|---------------|------------|
| Muskoka | Ontario, Canada | HIGH (~1km) | 60km |
| Santa Teresa | Costa Rica | LOW (~200m) | 15km |
| Toronto | Ontario, Canada | MEDIUM (~500m) | 30km |

## Infrastructure

- URL: `https://whosup.camerontora.ca`
- Backend: port 3001, container name `whosup-api`; nginx strips `/api` prefix and proxies to backend
- Docker Compose at `whosup/docker-compose.yaml`
- PostgreSQL on port 5433 (non-standard to avoid conflicts)
- iOS app connects to production URL: `https://whosup.camerontora.ca/api`
- **NOT behind OAuth2 Proxy** — public API with app-managed authentication (Apple/Google Sign In)
- Nginx rate limits: auth endpoints 5 req/min per IP, general API 10 req/sec per IP (burst 20)
- Security headers: X-Content-Type-Options, X-Frame-Options, X-XSS-Protection, Referrer-Policy
- Monitored by [[wiki/infrastructure/status-dashboard]] and [[wiki/infrastructure/health-api]] (container + port 3001 check)

## Deployment

Process manager: PM2 in cluster mode (`instances: 'max'`), auto-restart on crash, `max_memory_restart: '1G'`. Logs to `/var/log/whosup/`.

Database backups: `pg_dump` daily at 3 AM, 7-day retention. Stored at `/var/backups/whosup/`.

Zero-downtime deploy: `git pull` → `npm run build` → `npx prisma migrate deploy` → `pm2 reload whosup-api`.

## Key Files

| Path | Purpose |
|------|---------|
| `backend/src/routes/` | API endpoints |
| `backend/src/socket/` | Real-time WebSocket events |
| `backend/prisma/` | Database schema and migrations |
| `ios/WhosUp/Views/` | SwiftUI views by feature |
| `ios/WhosUp/Services/` | API client, location/auth managers |
| `docs/API.md` | Full API reference |
| `docs/ARCHITECTURE.md` | System design and data flow |

## Change Log

- 2026-01-31: Added Docker Compose for persistent backend deployment
- Earlier: Banner photo upload, Apple Maps navigation, group chat, connections, mark-guest-as-paid

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/whosup-repo]] — full architecture, iOS app setup, API reference, launch regions
- `raw/repos/whosup/docs/` — ARCHITECTURE.md, DATABASE.md, DEPLOYMENT.md (entity model, privacy layers, PM2 config)
