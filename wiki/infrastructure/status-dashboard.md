---
title: "Status Dashboard"
type: infrastructure
tags: [infra, monitoring, dashboard, gcp]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
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
| Frontend | React |

## Features

- Real-time service health for all `*.camerontora.ca` services
- System metrics panel (CPU, RAM, disk) via [[wiki/infrastructure/health-api]]
- Storage panel: HOMENAS RAID5 health, CAMRAID status, per-drive SMART data (expandable)
- Admin panel: SSH restart button with confirmation dialog and reboot progress tracking
- Reboot verification: polls for services to return, verifies HOMENAS and CAMRAID healthy before marking complete

## Key Design Decisions

- Hosted on GCP (not home server) so it's available during outages and internet drops
- `status.camerontora.ca` is a CNAME to GCP Cloud Run, not a home server subdomain — SSL is managed by Google, not Let's Encrypt

## Connected Projects

- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/infrastructure/health-api]]
- [[wiki/infrastructure/storage-raid]]

## Sources

- [[wiki/sources/infrastructure-repo]] — dashboard architecture, severity levels, admin features
