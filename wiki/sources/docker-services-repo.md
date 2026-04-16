---
title: "Source: docker-services repo"
type: source
tags: [source, media, plex, vpn, transmission, arr]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: docker-services repo

**Raw file:** `raw/repos/docker-services`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

The docker-services repo is the Docker Compose stack for all home media services. It runs Plex and the full *arr automation suite (Radarr, Sonarr, Jackett, Bazarr, Tdarr, FlareSolverr, Recyclarr), plus Tautulli for analytics and Transmission for torrenting behind a PIA WireGuard VPN.

The VPN setup is more sophisticated than the infrastructure repo revealed: three Gluetun containers (Toronto, Montreal, Vancouver) run simultaneously, and Transmission connects to one at a time via Docker's `network_mode`. A systemd watcher service automatically recreates Transmission whenever a Gluetun container restarts — solving the container ID problem. All Gluetun containers have a 1GB memory cap due to a known memory leak when the VPN is unhealthy.

Overseerr has been replaced by Seerr (2026-03-28); Ombi is decommissioned. Watchmap is a separate repo (symlinked at `raw/repos/watchmap`) — its backend writes location data to `camerontora.ca/app/data/locations.json`.

## Key Facts / Data Points

- Active VPN: Toronto (`gluetun-toronto`, port 9091); Montreal=9092, Vancouver=9093
- Vancouver was 7x faster than Toronto in Jan 2026 speed tests but had DNS issues — switched back to Toronto 2026-01-12
- All gluetun containers capped at 1GB RAM (memory leak when VPN unhealthy)
- Gluetun-watcher systemd service: `/etc/systemd/system/gluetun-watcher.service`
- Storage: `/HOMENAS` (primary), `/HOMENAS2` (secondary), `/CAMRAID`; `/dev/shm` for Plex transcoding
- Recyclarr runs at 3:15am daily (quality profile sync for Sonarr/Radarr)
- Tdarr and Bazarr kept stopped (excluded from update command)
- Network declared as external (`docker-services_default`) to avoid Docker Compose label mismatch errors
- Watchmap backend writes to `camerontora.ca/app/data/locations.json` (bind-mounted read-only into the container)

## Relevance to Wiki

Fills in the project-level detail for the media stack. Adds significant depth to the Gluetun VPN page (3-region architecture, watcher service, memory limits, speed data not in the infrastructure repo).

## Contradictions or Tensions

- Infrastructure repo described a single Gluetun container (`gluetun-toronto`). This repo reveals three run in parallel. Updated [[wiki/infrastructure/gluetun-vpn]] accordingly.

## Pages Updated

- [[wiki/projects/docker-services]] — created
- [[wiki/projects/plex-media-server]] — created
- [[wiki/projects/arr-suite]] — created
- [[wiki/projects/transmission]] — created
- [[wiki/infrastructure/gluetun-vpn]] — updated with 3-region architecture, watcher, memory limit, speed data
