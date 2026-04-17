---
title: "Status Dashboard"
type: infrastructure
tags: [infra, monitoring, dashboard, gcp]
created: 2026-04-16
updated: 2026-04-17
source_count: 4
---

# Status Dashboard

A GCP-hosted status page that displays health and metrics for all home server services. Hosted externally so it remains accessible even when the home server is down.

## Specification

| Property | Value |
|----------|-------|
| URL | `https://status.camerontora.ca` |
| Hosting | GCP Cloud Run (CNAME, SSL managed by Google) |
| Source | `infrastructure/status-dashboard/` |
| Backend | Python/Flask |
| Frontend | Preact + Tailwind |
| GCP project | `cameron-tora` / `us-central1` |
| Deploy | GitHub Actions → Cloud Run |

## Services Monitored (14)

### Public
| Service | URL |
|---------|-----|
| Main Site | `camerontora.ca` |
| Plex | `plex.camerontora.ca` |
| Seerr | `seerr.camerontora.ca` |

### Protected (OAuth)
| Service | URL |
|---------|-----|
| Haymaker | `haymaker.camerontora.ca` |
| Watchmap | `watchmap.camerontora.ca` |
| Radarr | `radarr.camerontora.ca` |
| Sonarr | `sonarr.camerontora.ca` |
| Jackett | `jackett.camerontora.ca` |
| Tautulli | `tautulli.camerontora.ca` |
| Transmission | `transmission.camerontora.ca` |
| Netdata | `netdata.camerontora.ca` |

### API
| Service | URL |
|---------|-----|
| Health API | `health.camerontora.ca/api/health/ping` |
| Who's Up API | `whosup.camerontora.ca/api/health` |
| SBA API | `sba.camerontora.ca/api/health` |

## Health Check Logic

HTTP responses are interpreted as:

| Response | Status |
|----------|--------|
| 2xx or 3xx | **Up** |
| **401** | **Up** — OAuth services return 401 unauthenticated; proves service is running |
| 4xx/5xx (except 401) | **Down** |
| Timeout (15s) | **Down** |

Each service gets both an external check (from GCP) and an internal check (container running + local port responding via [[wiki/infrastructure/health-api]]). The two-level result:

| External | Internal | Meaning |
|----------|----------|---------|
| Up | Up | Operational |
| Down | Up | Network/nginx/DNS issue — service works internally |
| Down | Down | Service itself is down |

## Metrics Sources

**Real-time (every 10s)** — Netdata public endpoints at `netdata.camerontora.ca/api/metrics/cpu` and `/api/metrics/ram` (OAuth bypassed for these specific charts only):
- CPU and RAM shown with "Live" indicator

**Periodic (every 5 min)** — [[wiki/infrastructure/health-api]]:
- Load average, per-disk usage, speed test results, Plex library count
- Falls back to health-api CPU/RAM values with "Using cached data" warning if Netdata unavailable

## Overall Status Logic

| Condition | Status |
|-----------|--------|
| Health API unreachable | `unhealthy` (red) |
| More than 3 services down | `unhealthy` (red) |
| 1–3 services down | `degraded` (yellow) |
| All services up | `healthy` (green) |

## Admin Panel

Visible to authenticated users — detected via `health.camerontora.ca/api/admin/whoami` on page load; an existing `_oauth2_proxy` cookie from any protected service is sufficient.

### VPN Switching
Switch Transmission between Toronto/Montreal/Vancouver with one click. Triggers the full switch process: recreate container, wait for ready, update Sonarr/Radarr ports, update nginx config.

### Container Restart
Each service card shows a restart button (admin only). Click once → confirm → backend triggers `docker restart` asynchronously. Card shows spinner; uptime counter turns green for 5 minutes after restart to confirm it worked.

### Server Reboot
Red "Restart" button in the System Metrics panel. Multi-phase RebootDialog:

| Phase | Display |
|-------|---------|
| `confirm` | "Are you sure?" modal with Cancel/Restart |
| `rebooting` | Live service grid with red/green status dots, progress bar, elapsed timer |
| `complete` | All services green + HOMENAS and CAMRAID arrays verified healthy |

Polls GCP `/api/status` every 5s during reboot (home server is offline, so GCP is the source of truth). Times out after 5 minutes (60 attempts × 5s). Health-api runs with `privileged: true` + `pid: host`; reboot command: `nsenter -t 1 -m -u -i -n -- reboot` (enters host's PID 1 namespace from the container). A 2-second delay before reboot ensures the HTTP response is sent first.

## Historical Data (Firestore)

Status snapshots stored every 5 minutes to GCP Firestore (Native mode, `us-central1`):
- Service status (up/down), response times, system metrics, speed test results
- Retention: 7 days
- Uptime history panel: colored bars per service over 24h or 7-day views
- API: `GET /api/history?hours=24&service=Plex`

## Architecture

```
Cloud Scheduler (*/5 min)
    → POST /api/check → Cloud Run status-dashboard
        ├── Health checks all 15 services
        ├── Pulls metrics from health-api + Netdata
        ├── Reads DNS state from GoDaddy API
        └── Sends Discord alerts on state changes
```

## Wiki Q&A Panel

Natural-language Q&A panel backed by the camwiki knowledge base. Public-facing — no authentication required.

| Property | Value |
|----------|-------|
| Endpoint | `POST /api/wiki-qa` |
| Auth | None (public) |
| Model | `claude-sonnet-4-6` |
| Max tokens | 600 |
| Answer format | Markdown, rendered via `marked` |

**How it works:**
- On Cloud Run startup, backend fetches `wiki_context.txt` from GCS bucket `camwiki-context` into memory
- Background thread re-fetches every 3600s — wiki changes propagate within 1 hour without a redeploy
- Home server runs an hourly cron: `upload_wiki_context.py` reads `wiki/**/*.md` (skipping `sources/`) and uploads to GCS
- `ANTHROPIC_API_KEY` stored in GCP Secret Manager, injected at runtime (not a plain env var)
- Cloud Run memory bumped to 512Mi to accommodate `anthropic` + `google-cloud-storage` dependencies

**Divergence from original spec:** Originally designed as admin-only (`adminAuth` guard). Shipped as fully public — anyone visiting `status.camerontora.ca` can query it.

Upload script: `/home/camerontora/camwiki/scripts/upload_wiki_context.py`
Full implementation spec: `infrastructure/docs/WIKI-QA-FEATURE.md`

## Plex Platform Banner

On every `/api/status` poll the backend calls `https://status.plex.tv/api/v2/summary.json` (Atlassian Statuspage public API) in parallel with all other health checks. When Plex reports an active incident the frontend renders a banner above the failover banner:

| Plex `indicator` | Banner colour |
|-----------------|---------------|
| `none` | Hidden |
| `minor` | Amber |
| `major` / `critical` | Red |

**Banner content:** Actual incident title + current status (e.g. "Issues with plex.tv API — Investigating"), page-level description as subtitle, and a link to `status.plex.tv`. Distinguishes "Plex the company is having issues" from "our server is down."

Backend field: `plex_platform: { indicator, description, incidents[] }` in `/api/status` response.
Frontend component: `PlexStatusBanner.jsx`.

## Change Log

- 2026-04-17: Wiki Q&A panel added (public, GCS-backed, markdown rendering, Secret Manager for API key)
- 2026-04-16: Plex platform status banner added; jshor96@aol.com granted OAuth access

## Key Design Decisions

- Hosted on GCP so it's always reachable during home server or internet outages — see [[wiki/decisions/infra/decision-status-dashboard-on-gcp]]
- `status.camerontora.ca` is a CNAME (not an A record) — SSL managed by Google, not Let's Encrypt; must not be added to the `camerontora-services` cert
- Doubles as the DNS failover landing page: when `@`, `plex`, `seerr` flip to GCP IPs, visitors see this dashboard — see [[wiki/concepts/dns-failover]]

## Connected Projects

- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/infrastructure/health-api]]
- [[wiki/infrastructure/storage-raid]]
- [[wiki/concepts/dns-failover]]
- [[wiki/decisions/infra/decision-status-dashboard-on-gcp]]

## Sources

- [[wiki/sources/infrastructure-repo]] — STATUS-DASHBOARD.md, MONITORING.md, full dashboard architecture and admin panel implementation
