---
title: "Docker Services"
type: project
tags: [project, media, docker, plex, vpn]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Docker Services

The home media server Docker Compose stack. Runs Plex and all supporting services for media acquisition, management, and monitoring.

## Status

Current status: `active`
Last meaningful change: 2026-03-28 (Overseerr → Seerr migration)

## Architecture

All services share the `docker-services_default` network (declared external to avoid Docker Compose label mismatch). Transmission is the exception — it runs inside a Gluetun container's network namespace.

| Service | Port | Role | State |
|---------|------|------|-------|
| Plex | 32400 | Media server | active |
| Tautulli | 8181 | Plex analytics | active |
| Radarr | 7878 | Movie management | active |
| Sonarr | 8989 | TV management | active |
| Jackett | 9117 | Torrent indexer proxy | active |
| FlareSolverr | 8191 | Cloudflare bypass for Jackett | active |
| Transmission | 9091 | Torrent client (via VPN) | active |
| Seerr | 5055 | Media requests | active |
| Recyclarr | — | Quality profile sync (3:15am daily) | active |
| Bazarr | 6767 | Subtitle management | stopped |
| Tdarr | 8265 | Media transcoding | stopped |
| Ombi | 3579 | Media requests (legacy) | decommissioned |
| Watchmap | 5080 | Live Plex stream map | active (separate repo) |
| gluetun-toronto | 9091/8090 | VPN (active) | active |
| gluetun-montreal | 9092/8091 | VPN (idle) | active/unhealthy |
| gluetun-vancouver | 9093/8092 | VPN (idle) | active/unhealthy |

## Infrastructure

- Host: [[wiki/infrastructure/home-server]]
- Reverse proxy: [[wiki/infrastructure/nginx-reverse-proxy]] (protected services require OAuth2)
- VPN: [[wiki/infrastructure/gluetun-vpn]]
- Docker network: [[wiki/infrastructure/docker-networks]] (`docker-services_default`, 172.19.0.0/16)

## Storage

| Mount | Purpose |
|-------|---------|
| `/HOMENAS` | Primary NAS — media library and torrents |
| `/HOMENAS2` | Secondary NAS |
| `/CAMRAID` | RAID array |
| `/dev/shm` | RAM disk for Plex transcoding |

## Key Decisions

- [[wiki/projects/transmission]] runs inside Gluetun's network namespace — all torrent traffic exits through VPN
- Three Gluetun regions run in parallel; Transmission uses one at a time (see [[wiki/infrastructure/gluetun-vpn]])
- Bazarr and Tdarr kept stopped — excluded from update command to avoid unnecessary restarts
- Network declared as external in docker-compose.yaml to avoid label mismatch (pre-dates Docker Compose label requirements)

## Change Log

- 2026-03-28: Overseerr replaced by Seerr; Ombi decommissioned
- 2026-01-12: Switched active VPN from Vancouver back to Toronto (Vancouver DNS issues)
- 2026-01-11: Added 1GB memory limit to all Gluetun containers (memory leak fix)
- 2026-01-11: Added gluetun-watcher systemd service (auto-recreate Transmission on Gluetun restart)
- 2026-01-11: Added multi-region VPN support (Toronto, Montreal, Vancouver)

## Updating Services

```bash
cd /home/camerontora/docker-services
docker-compose pull
docker-compose up -d plex tautulli transmission sonarr radarr jackett overseerr ombi flaresolverr recyclarr tdarr bazarr
docker image prune -f
```

Excluded from update: `gluetun-*` (VPN state), `watchmap-*` (built from local Dockerfile).

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/docker-services-repo]] — full stack config, VPN architecture, update procedures
