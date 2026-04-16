---
title: "Arr Suite"
type: project
tags: [project, media, radarr, sonarr, jackett, bazarr, recyclarr]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Arr Suite

The media automation pipeline: finds, downloads, and organizes movies and TV shows. Radarr and Sonarr manage the libraries; Jackett proxies torrent indexers; Recyclarr keeps quality profiles synced; FlareSolverr bypasses Cloudflare on indexers; Bazarr manages subtitles (kept stopped).

## Status

Current status: `active` (Bazarr: stopped)
Last meaningful change: 2026-01-13

## Services

| Service | Port | Role | State |
|---------|------|------|-------|
| Radarr | 7878 | Movie library management and automation | active |
| Sonarr | 8989 | TV show library management and automation | active |
| Jackett | 9117 | Torrent indexer aggregation proxy | active |
| FlareSolverr | 8191 | Cloudflare challenge bypass for Jackett | active |
| Recyclarr | — | Syncs quality profiles from TRaSH Guides (3:15am daily) | active |
| Bazarr | 6767 | Subtitle download and management | stopped |

## Architecture

```
Seerr (requests)
    → Radarr / Sonarr (monitor + search)
        → Jackett (indexer proxy)
            → FlareSolverr (Cloudflare bypass)
        → Transmission (download via VPN)
    → Plex (library update on completion)
```

Recyclarr runs independently on a cron schedule, pushing quality profile configs from TRaSH Guides into Radarr and Sonarr.

## Infrastructure

- All services on `docker-services_default` network
- Download client: [[wiki/projects/transmission]] (Transmission)
- Media stored on `/HOMENAS`
- Protected by OAuth2 at `radarr.camerontora.ca`, `sonarr.camerontora.ca`, `jackett.camerontora.ca`

## Integrations

- **[[wiki/projects/transmission]]** — Radarr and Sonarr push downloads to Transmission; port must match active VPN location
- **[[wiki/projects/plex-media-server]]** — Radarr/Sonarr trigger Plex library scans on completion
- **[[wiki/infrastructure/gluetun-vpn]]** — VPN auto-failover updates Radarr and Sonarr download client ports via API after switching (30s wait loop)

## Key Decisions

- Bazarr and Tdarr kept stopped and excluded from the update command — not needed day-to-day, avoids unnecessary restarts
- Recyclarr uses TRaSH Guides to maintain consistent quality profiles — config synced from `recyclarr/` dir (contains nested git repos, excluded from main git tracking)

## Change Log

- 2026-01-13: VPN auto-failover now updates Radarr/Sonarr download client ports post-switch

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/docker-services-repo]] — service config, pipeline architecture, Recyclarr setup
