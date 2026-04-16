---
title: "Health API"
type: infrastructure
tags: [infra, api, monitoring, health]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Health API

A Flask container running on the home server that exposes system metrics to external consumers — primarily the [[wiki/infrastructure/gcp-external-monitoring]] system.

## Specification

| Property | Value |
|----------|-------|
| Container | `health-api` |
| Port | 5000 |
| URL | `https://health.camerontora.ca` |
| Source | `infrastructure/health-api/` |
| Auth | `X-API-Key` header (required for `/api/health`) |

## Endpoints

| Endpoint | Auth | Description |
|----------|------|-------------|
| `GET /api/health/ping` | None | Liveness check, returns `{"status": "ok"}` |
| `GET /api/health` | X-API-Key | Full metrics: CPU, RAM, disk, Plex, speedtest |
| `GET /api/health/vpn/switch` | X-API-Key | Triggers VPN failover |
| `GET /` | None | Service info |

## What It Reads

- `/proc` — CPU and RAM usage
- Disk mounts — usage percentages for all mounted volumes
- Plex API — library availability
- `/var/lib/health-api/speedtest.json` — latest speedtest result (written by cron every 30 min)
- `/proc/mdstat` — HOMENAS software RAID5 health and sync status
- `smartctl` — per-drive SMART data for all 8 HOMENAS drives (sdc–sdj)

## Connected Projects

- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/infrastructure/storage-raid]]
- [[wiki/infrastructure/gluetun-vpn]]

## Sources

- [[wiki/sources/infrastructure-repo]] — API endpoints, SMART monitoring, speedtest integration
