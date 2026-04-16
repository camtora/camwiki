---
title: "Who's Up"
type: project
tags: [project, social, ios, swift, nodejs, postgis, maps]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Who's Up

Map-first social app for discovering and joining nearby activities. Users broadcast that they're available ("up for surfing, coffee, hiking") and discover others doing the same within a geographic area. Hosted at `whosup.camerontora.ca`.

## Status

Current status: `active`
Last meaningful change: 2026-03-31 (reverted login tagline)

## Architecture

| Component | Tech |
|-----------|------|
| Backend | Node.js 20, Express, TypeScript |
| Database | PostgreSQL 15+ with PostGIS (geospatial queries) |
| ORM | Prisma 5.22 |
| Real-time | Socket.io 4.8 (WebSocket events) |
| Auth | JWT + Apple Sign In + Google Sign In |
| Validation | Zod |
| iOS app | SwiftUI, iOS 17+, Xcode 15+ |
| iOS auth | Sign in with Apple |
| iOS maps | MapKit + Core Location |
| Deployment | Docker Compose |

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
