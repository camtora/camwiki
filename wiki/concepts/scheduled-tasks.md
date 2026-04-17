---
title: "Scheduled Tasks"
type: concept
tags: [concept, automation, cron, scheduler]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Scheduled Tasks

Everything that runs automatically across the home server, GCP, and application layers.

## Home Server Cron Jobs

These run on the Ubuntu server (`/etc/cron.d/` unless noted).

| Job | Schedule | Script | Output / Notes |
|-----|----------|--------|----------------|
| GoDaddy DDNS update | Every 10 min | `infrastructure/scripts/godaddy-ddns.sh` | Skips update if `@` already points to GCP failover IP. Log: `/var/log/godaddy-ddns.log` |
| Speed test | Every 30 min | `infrastructure/scripts/speedtest.sh` | Output: `/var/lib/health-api/speedtest.json`. 90s grace period after Gluetun restart. |
| VNC maintenance restart | Daily at 4 AM | `/etc/cron.d/vnc-maintenance` | Fallback restart of RealVNC to clear stale cloud sessions |
| camwiki → GCS upload | Hourly (`0 * * * *`) | `/home/camerontora/camwiki/scripts/upload_wiki_context.py` | Reads `wiki/**/*.md`, uploads to `gs://camwiki-context/wiki_context.txt` for the status dashboard Q&A panel |
| camerontora.ca location backfill | Hourly (`0 * * * *`) | `camerontora.ca/scripts/backfill-locations.ts` | Geolocates new Tautulli viewer IPs; writes to file-based location cache |
| Haymaker DB backup | Daily at 3 AM | `pg_dump` | 7-day retention. See [[wiki/projects/haymaker]] |

## GCP Cloud Scheduler

Triggers Cloud Run services via HTTP POST.

| Job | Schedule | Target | Effect |
|-----|----------|--------|--------|
| `home-health-check` | Every 5 min (`*/5 * * * *`) | `POST /api/check` on `status-dashboard` | Health checks all 14 services, pulls metrics, writes Firestore snapshot, sends Discord alerts |
| GCP external monitor | Every 5 min (`*/5 * * * *`) | `home-monitor` Cloud Run | External availability check; triggers DNS failover and VPN failover logic |

## In-Process Background Threads

These run inside long-lived services, not as OS-level crons.

| Service | Interval | What it does |
|---------|----------|--------------|
| `status-dashboard` Cloud Run | Every 3600s | Re-fetches `wiki_context.txt` from GCS — keeps Q&A knowledge base current without a redeploy |
| `watchmap` backend | Every 2s | Polls Tautulli API for active streams; broadcasts `stream_update` via Socket.IO to all connected clients |
| `watchmap` backend | Every 2s | Reads host `/proc` files (via `/host/proc` mount) for CPU/RAM/network; broadcasts `system_stats_update` via Socket.IO |

## Application-Level Scheduled Jobs

Crons managed inside application containers (not OS-level).

| App | Scheduler | Job | Schedule | Effect |
|-----|-----------|-----|----------|--------|
| Recyclarr | Docker container cron | Push TRaSH quality profiles | Daily | Applies TRaSH Guides templates to Radarr + Sonarr quality profiles automatically |
| SBCA | `node-cron` | `checkDuesExpiry` | Daily | Sends push renewal reminders to members with expiring dues |
| SBCA | `node-cron` | `syncWaterLevels` | Daily at 8 AM ET | Polls Environment Canada MSC GeoMet API; stores results in PostgreSQL |
| Haymaker | Internal cron | Discord habit reminders | Configured per-task | Fires Discord reminder if a tracked task is incomplete by its deadline |

## Persistent Services

Long-running processes managed by systemd (not cron-based).

| Service | Unit | Port | Notes |
|---------|------|------|-------|
| Quartz Wiki | `quartz-wiki` | 3004 | Serves `wiki.camerontora.ca`; watches content for changes and rebuilds automatically |

## Frontend Polling

Client-side intervals — not server crons, but relevant for understanding system load.

| Client | Interval | What it polls |
|--------|----------|---------------|
| Status dashboard | Every 10s | Netdata CPU/RAM endpoints |
| Status dashboard | Every 5 min | `/api/status` (GCP Cloud Run) |
| Watchmap tvOS app | Every 5s | `/api/locations`, `/api/streams`, `/api/metrics` |

## Related Pages

- [[wiki/infrastructure/dns-ssl]] — GoDaddy DDNS script detail
- [[wiki/infrastructure/health-api]] — speed test cron and output format
- [[wiki/infrastructure/gcp-external-monitoring]] — Cloud Scheduler jobs and monitor logic
- [[wiki/infrastructure/status-dashboard]] — wiki context refresh thread, Q&A panel
- [[wiki/infrastructure/home-server]] — VNC maintenance cron
- [[wiki/projects/haymaker]] — DB backup cron
- [[wiki/projects/camerontora-ca]] — location backfill cron
- [[wiki/projects/arr-suite]] — Recyclarr
- [[wiki/projects/sbca]] — node-cron jobs
