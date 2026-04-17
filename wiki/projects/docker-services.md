---
title: "Docker Services"
type: project
tags: [project, media, docker, plex, vpn]
created: 2026-04-16
updated: 2026-04-17
source_count: 2
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
| gluetun-toronto | 9091/8090 | VPN region | active |
| gluetun-montreal | 9092/8091 | VPN region | active |
| gluetun-vancouver | 9093/8092 | VPN region | active |

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

- [[wiki/decisions/docker-services/transmission-gluetun-network-mode]] — Transmission runs inside Gluetun's network namespace; strictest VPN kill-switch, no leak possible
- [[wiki/decisions/docker-services/recyclarr-for-quality-profiles]] — Recyclarr syncs TRaSH Guides quality profiles into Radarr/Sonarr daily; config is version-controlled
- Three Gluetun regions run in parallel; GCP health checks auto-fail Transmission over to the next healthy region — see [[wiki/infrastructure/gluetun-vpn]]
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
docker-compose up -d plex tautulli transmission sonarr radarr jackett seerr flaresolverr recyclarr tdarr bazarr
docker image prune -f
```

Excluded from update:
- `gluetun-*` — restarting drops the VPN connection; only restart intentionally
- `watchmap-*` — built from local Dockerfile, `pull` does nothing; rebuild separately after source changes
- `bazarr`, `tdarr` — kept stopped; excluded to avoid unnecessary restarts

### Post-Update Validation

```bash
docker-compose ps  # all expected services show Up
```

Spot-check key services:
| Service | URL |
|---------|-----|
| Plex | http://localhost:32400/web |
| Transmission | http://localhost:9091 |
| Sonarr | http://localhost:8989 |
| Radarr | http://localhost:7878 |
| Seerr | http://localhost:5055 |

Expected non-healthy state: `gluetun-montreal` and `gluetun-vancouver` show `Up (unhealthy)` — they are running but idle; only the active gluetun container needs to be healthy.

## Known Issues

**High** _(mitigated)_
- **Gluetun memory leak** — Gluetun leaks memory on unhealthy VPN connections (DNS-over-TLS retry accumulation). Without mitigation, containers grow to 2+ GB. **Mitigated** via `mem_limit: 1g` on all three containers — they auto-restart when the limit is hit, resetting the leak. (commits `f428736`, Jan 2026)
- **Transmission-Gluetun network orphaning** — Transmission shares Gluetun's network namespace via container ID reference. When Gluetun restarts (due to memory limit or update), the container ID changes and Transmission loses connectivity. **Mitigated** via `gluetun-watcher.service` systemd service that auto-recreates Transmission when its active Gluetun restarts. (commit `d8675a0`, Jan 2026)

**Medium** _(accepted limitations)_
- **Vancouver Gluetun DNS issues** — `gluetun-vancouver` has broken DNS resolution making it unsuitable for active use. Transmission uses `gluetun-montreal` instead; Vancouver runs only as failover capacity. Do not switch Transmission to Vancouver without investigating.
- **Update command must exclude Gluetun** — running bare `docker-compose up -d` (without the explicit service list) would recreate Gluetun containers, drop the VPN, and leave Transmission without a network. Always use the explicit update command in the runbook above.
- **PIA port forwarding changes dynamically** — if PIA rotates the forwarded port, Transmission's configured port becomes stale and peers can no longer connect inbound. Requires manual port update in Transmission settings.

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/docker-services-repo]] — full stack config, VPN architecture, update procedures
