---
title: "SBCA — Stephens Bay Cottage Association"
type: project
tags: [project, mobile, expo, react-native, fastify, postgresql, stripe, accessibility]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# SBCA — Stephens Bay Cottage Association

Mobile app (React Native/Expo) for the Stephens Bay Cottage Association on Lake Muskoka, Ontario. Serves association members — primary demographic ages 55–75. **Accessibility is a first-class requirement** throughout. Hosted at `sba.camerontora.ca` and `*.sba.camerontora.ca`.

## Status

Current status: `active` (Phase 1 & 2 complete; Phase 3 admin portal in progress)
Last meaningful change: 2026-04-09 (nginx domain routing docs, metro.sba subdomain)

## Architecture

Monorepo:

```
sbca/
├── app/          ← React Native (Expo SDK 55, Expo Router)
├── backend/      ← Fastify API (Node 20, Drizzle ORM, PostgreSQL 16)
├── admin/        ← Admin web portal (Next.js 14) [Phase 3, in progress]
└── infrastructure/ ← Docker Compose (Postgres + MinIO + API)
```

API on port 3003 → nginx → `https://sba.camerontora.ca`
MinIO: API port 9010, console port 9011

### Mobile Stack

| Layer | Tech |
|-------|------|
| Framework | React Native, Expo SDK 55 |
| Routing | Expo Router (file-based) |
| State | Zustand |
| HTTP | Axios + JWT interceptors |
| Auth storage | Expo SecureStore |
| Offline cache | AsyncStorage |
| Payments | Stripe React Native SDK |
| Push notifications | Expo Push API (free, no FCM needed) |
| Icons | Ionicons (@expo/vector-icons) |

### Backend Stack

| Layer | Tech |
|-------|------|
| Runtime | Node.js 20, TypeScript |
| Framework | Fastify |
| ORM | Drizzle ORM |
| Database | PostgreSQL 16 |
| Object storage | MinIO (S3-compatible) — lake observation photos |
| Scheduled jobs | node-cron |

## Features

### Feature 1 — Membership Portal
- Member registration, login, JWT auth (access + refresh tokens)
- Annual dues payment via Stripe (CardField only; PaymentIntent created server-side — verified server-side to prevent client manipulation)
- Digital membership card with status and expiry
- Member directory with opt-in contact sharing
- Push notification 30 days before dues expire

### Feature 2 — Lake Health Dashboard
- Lake status indicator (Safe / Caution / Advisory) — uses emoji + colour + text, **never colour alone** (accessibility)
- Community lake observation submissions with photo upload to MinIO
- Admin approval queue before observations go public
- Water level feed from Environment Canada MSC GeoMet API (synced daily at 8 AM ET)
  - Lake Muskoka at Beaumaris: station `02EB018`
  - South Branch Muskoka River at Baysville: station `02EB008`
- 30-day water level chart with seasonal context labels
- Push notifications on significant level changes (±30 cm/week)

### Feature 3 — Events & Community Board
- Events list with category filter, RSVP, 48-hour reminder notification
- Community noticeboard with category posts and threaded replies
- Admin moderation — posts and lake reports require approval before going public

## Infrastructure

- Domains: `sba.camerontora.ca`, `admin.sba.camerontora.ca`, `metro.sba.camerontora.ca` (wildcard `*.sba` cert)
- Docker network: `infrastructure_default` (nginx routes to backend)
- Reverse proxy: [[wiki/infrastructure/nginx-reverse-proxy]] (`docs/04-sba.conf`)
- SSL: shared `camerontora-services` Let's Encrypt cert (see [[wiki/infrastructure/dns-ssl]])
- iOS build requires `ios/WhosUp.xcworkspace` (not `.xcodeproj`) — use `xcworkspace` after `expo prebuild`

## Key Files

| Path | Purpose |
|------|---------|
| `backend/src/` | Fastify routes, Drizzle schema, scheduled jobs |
| `backend/prisma/` | DB migrations |
| `app/app/` | Expo Router pages by feature |
| `infrastructure/docker-compose.yaml` | Postgres + MinIO + API containers |
| `docs/` | Developer docs, API reference, deployment guide |

## Change Log

- 2026-04-09: nginx domain routing documented; metro.sba subdomain added
- Earlier: Phase 3 admin portal (Next.js 14) scaffolded and brought live
- Earlier: Stripe server-side payment verification; station IDs verified

## Open Questions

- Phase 3 admin portal — scope and completion status unclear from commits

## Sources

- [[wiki/sources/sbca-repo]] — full monorepo architecture, Expo/Fastify setup, lake data sources, accessibility requirements
