---
title: "GCP External Monitoring"
type: infrastructure
tags: [infra, monitoring, gcp, cloud-run, discord, alerts]
created: 2026-04-16
updated: 2026-04-16
source_count: 2
---

# GCP External Monitoring

An external monitoring system running on Google Cloud Platform that checks the home server from outside and alerts via Discord. Solves the fundamental limitation of local monitors: they can't alert when the home internet is down.

## Specification

| Property | Value |
|----------|-------|
| GCP project | `cameron-tora` |
| Service | Cloud Run `home-monitor`, region `us-central1` |
| Schedule | Every 5 minutes (`*/5 * * * *`) via Cloud Scheduler job `home-health-check` |
| Alerts | Discord webhook (URL in GCP Secret Manager: `discord-webhook-url`) |
| Source | `infrastructure/gcp-monitor/` |
| Deployment | GitHub Actions (auto-deploys on push to main) |

## What Gets Checked

| Endpoint | Purpose |
|----------|---------|
| `https://camerontora.ca` | Home server reachability |
| `https://status.camerontora.ca` | Status dashboard reachability |
| `https://health.camerontora.ca/api/health` | Full system metrics (CPU, RAM, disk, speed, Plex) |
| `https://plex.camerontora.ca/library/sections` | Plex library availability |

## Alert Thresholds

| Metric | Threshold | Alert |
|--------|-----------|-------|
| CPU | > 90% | Degraded |
| RAM | > 95% | Degraded |
| Disk `/` | > 95% | Degraded |
| Disk `/home` | > 95% | Degraded |
| Disk `/var` | > 90% | Degraded |
| Disk `/CAMRAID` | > 95% | Degraded |
| Disk `/HOMENAS` | > 95% | Degraded |
| Upload speed | < 5 Mbps | Degraded |
| Speed test age | > 2 hours | Degraded (speedtest cron may be broken) |
| Plex libraries | = 0 | Major (storage mount likely missing) |

## Alert Severity Levels

| Level | Color | Triggers |
|-------|-------|----------|
| Major (🔴) | Red | Home internet down, Plex down, HOMENAS RAID unhealthy |
| Minor (🟠) | Orange | Active VPN unhealthy, service down, VPN failover failed |
| Degraded (🟡) | Yellow | Threshold breach (CPU/disk/speed), SSL cert expiring, VPN failover starting |
| Recovery (✅) | Green | Service restored after alert |

**Alert deduplication:** only alerts on state changes — first failure triggers alert; subsequent failures while still failing do not re-alert. Recovery triggers a "back online" notification.

## VPN Health Alerting

All three VPN locations (Toronto/Montreal/Vancouver) are monitored, but **alerts are only sent for the active VPN**. Inactive containers that are unhealthy do not trigger Discord notifications — they don't affect current operations and their `Up (unhealthy)` state is expected.

## Additional Capabilities

**DNS Failover** — when home internet is detected as down, triggers a GoDaddy API call to flip `@`, `plex`, `seerr` A records to GCP static IP (`216.239.32.21`). Reverts automatically on recovery. See [[wiki/concepts/dns-failover]] and [[wiki/decisions/infra/decision-dns-failover-approach]].

**VPN Auto-Failover** — tracks consecutive unhealthy checks on the active VPN. After 6 checks (~30 min), calls `health.camerontora.ca/api/health/vpn/switch` to switch to the healthiest available PIA server (selected by download speed). Discord notification on start, completion, or failure. See [[wiki/infrastructure/gluetun-vpn]].

**Concurrent health checks** — runs all checks concurrently to stay within Cloud Run's 60s request timeout.

## Architecture

```
Cloud Scheduler (*/5 min)
    → POST /check → Cloud Run home-monitor
        ├── GET camerontora.ca
        ├── GET status.camerontora.ca
        ├── GET health.camerontora.ca/api/health (X-API-Key)
        └── GET plex.camerontora.ca/library/sections (X-Plex-Token)
    → Discord webhook (on failure or recovery)
    → GoDaddy API (on internet outage detected)
    → health-api /vpn/switch (on VPN unhealthy 30+ min)
```

## GCP Secrets

| Secret | Used for |
|--------|---------|
| `discord-webhook-url` | Alert notifications |
| `health-api-key` | Authenticating `/api/health` requests |
| `plex-token` | Authenticating Plex library check |
| `gcp-static-ip` | IP written to DNS during failover (`216.239.32.21`) |

## Connected Projects

- [[wiki/infrastructure/health-api]]
- [[wiki/infrastructure/status-dashboard]]
- [[wiki/infrastructure/gluetun-vpn]]
- [[wiki/concepts/dns-failover]]
- [[wiki/decisions/infra/decision-gcp-external-monitoring]]

## Sources

- [[wiki/sources/infrastructure-repo]] — MONITORING.md, full monitoring architecture, alert thresholds, VPN failover logic
