---
title: "GCP External Monitoring"
type: infrastructure
tags: [infra, monitoring, gcp, cloud-run, discord, alerts]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# GCP External Monitoring

An external monitoring system running on Google Cloud Platform that checks the home server from outside and alerts via Discord. Solves the fundamental limitation of local monitors: they can't alert when the home internet is down.

## Specification

| Property | Value |
|----------|-------|
| GCP project | `cameron-tora` |
| Service | Cloud Run `home-monitor`, region `us-central1` |
| Schedule | Every 5 minutes (`*/5 * * * *`) via Cloud Scheduler job `home-health-check` |
| Alerts | Discord webhook |
| Source | `infrastructure/gcp-monitor/` |
| Deployment | GitHub Actions (auto-deploys on push to main) |

## What Gets Checked

| Endpoint | Purpose |
|----------|---------|
| `https://camerontora.ca` | Home server reachability |
| `https://status.camerontora.ca` | Status dashboard |
| `https://health.camerontora.ca/api/health` | System metrics (CPU, RAM, disk, speed) |
| `https://plex.camerontora.ca/library/sections` | Plex availability |

## Alert Severity Levels

| Level | Color | Triggers |
|-------|-------|----------|
| Major | Red | Home internet down, Plex down, HOMENAS RAID unhealthy |
| Minor | Orange | Other services down (Radarr, Sonarr, etc.) |
| Degraded | Yellow | CPU >90%, RAM >95%, upload speed <5 Mbps |
| Healthy | Green | All systems operational, sent on recovery |

## Additional Capabilities

**DNS Failover** — when home internet is detected as down, triggers a GoDaddy DNS switch for `@`, `plex`, `seerr` to the GCP static IP (`216.239.32.21`). Reverts automatically on recovery. See [[wiki/concepts/dns-failover]].

**VPN Auto-Failover** — tracks consecutive unhealthy VPN checks. After 6 checks (~30 min) of unhealthy VPN, calls `health.camerontora.ca/api/health/vpn/switch` to trigger a failover to the healthiest available PIA server. See [[wiki/infrastructure/gluetun-vpn]].

**Concurrent health checks** — runs checks concurrently to stay within Cloud Run's 60s request timeout.

## Architecture

```
Cloud Scheduler (*/5 min)
    → POST /check → Cloud Run home-monitor
        ├── GET camerontora.ca
        ├── GET status.camerontora.ca
        ├── GET health.camerontora.ca/api/health (X-API-Key)
        └── GET plex.camerontora.ca/library/sections
    → Discord webhook (on failure or recovery)
```

## Connected Projects

- [[wiki/infrastructure/health-api]]
- [[wiki/infrastructure/status-dashboard]]
- [[wiki/infrastructure/gluetun-vpn]]
- [[wiki/concepts/dns-failover]]

## Sources

- [[wiki/sources/infrastructure-repo]] — full monitoring architecture, alert thresholds, VPN failover logic
