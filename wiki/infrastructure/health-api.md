---
title: "Health API"
type: infrastructure
tags: [infra, api, monitoring, health]
created: 2026-04-16
updated: 2026-04-16
source_count: 2
---

# Health API

A Flask container running on the home server that exposes system metrics to external consumers — primarily the [[wiki/infrastructure/gcp-external-monitoring]] system and the [[wiki/infrastructure/status-dashboard]].

## Specification

| Property | Value |
|----------|-------|
| Container | `health-api` |
| Port | 5000 |
| URL | `https://health.camerontora.ca` |
| Source | `infrastructure/health-api/` |
| Auth | `X-API-Key` header (for most endpoints); OAuth2 Proxy (for `/api/admin/*`) |
| Privileges | `privileged: true`, `pid: host` (required for server reboot via `nsenter`) |

## Endpoints

### Public (no auth)
| Endpoint | Description |
|----------|-------------|
| `GET /api/health/ping` | Liveness check, returns `{"status": "ok"}` |
| `GET /` | Service info and endpoint list |

### Authenticated (X-API-Key header)
| Endpoint | Description |
|----------|-------------|
| `GET /api/health` | Full system metrics: CPU, RAM, disk, load, Plex, speedtest, VPN status |
| `GET /api/health/public-ip` | Current home server public IP — used by status dashboard for DNS failback |
| `GET /api/health/services` | Internal service status: container running + local port responding per service |
| `POST /api/health/vpn/switch` | Trigger VPN failover (called by GCP monitor after 30min unhealthy) |

### Admin (OAuth2 Proxy — `X-Forwarded-Email` via nginx)
| Endpoint | Description |
|----------|-------------|
| `GET /api/admin/whoami` | Check authentication; used by dashboard to show/hide admin panel |
| `GET /api/admin/vpn/status` | VPN location status + active location |
| `POST /api/admin/vpn/switch` | Manual VPN switch (triggered from status dashboard UI) |
| `POST /api/admin/container/restart` | Async container restart |
| `POST /api/admin/server/reboot` | Async server reboot via `nsenter -t 1 -m -u -i -n -- reboot` |

## What It Reads

- `/proc` — CPU, RAM, and load average
- Disk mounts — usage percentages for `/`, `/home`, `/var`, `/CAMRAID`, `/HOMENAS`
- Plex API — library availability and count
- `/var/lib/health-api/speedtest.json` — latest speedtest result (written by cron every 30 min)
- `/proc/mdstat` — HOMENAS software RAID5 health and sync status
- `smartctl` — per-drive SMART data for all 8 HOMENAS drives (sdc–sdj)

## Full Health Response (summary)

```json
{
  "timestamp": "...",
  "cpu_percent": 45.2,
  "load": { "load_1m": 1.5, "load_5m": 1.2, "cpu_count": 6 },
  "memory": { "percent": 50.4, "used_gb": 15.7, "total_gb": 31.2 },
  "disk": {
    "/":       { "percent": 22.1 },
    "/home":   { "percent": 77.2 },
    "/CAMRAID":{ "percent": 59.8 },
    "/HOMENAS":{ "percent": 90.7 }
  },
  "plex": { "reachable": true, "library_count": 22 },
  "speed_test": {
    "home": { "download_mbps": 150.2, "upload_mbps": 25.4 },
    "vpn":  { "toronto": { "download_mbps": 80.5 } }
  }
}
```

## Services Response (`/api/health/services`)

Returns per-service container and local port status — used by the status dashboard for two-level health check display:

```json
{
  "services": [
    {
      "name": "Plex",
      "container": { "name": "plex", "running": true },
      "local_port": { "port": 32400, "responding": true, "status_code": 200 }
    }
  ]
}
```

## Speed Test Cron

Script: `/home/camerontora/infrastructure/scripts/speedtest.sh`
Cron: `/etc/cron.d/speedtest`, every 30 minutes
Output: `/var/lib/health-api/speedtest.json`

Tests: home internet + all three Gluetun containers (Toronto/Montreal/Vancouver) concurrently. The script also detects orphaned Transmission containers (see [[wiki/infrastructure/gluetun-vpn]]) and auto-repairs by calling `/api/health/vpn/switch`.

The speedtest script has a **90-second grace period** after any Gluetun restart: DNS takes 30–60s to initialize after container start, and running a connectivity test during that window caused false health failures.

## Connected Projects

- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/infrastructure/status-dashboard]]
- [[wiki/infrastructure/storage-raid]]
- [[wiki/infrastructure/gluetun-vpn]]

## Sources

- [[wiki/sources/infrastructure-repo]] — MONITORING.md, STATUS-DASHBOARD.md, full API endpoints and response schemas
