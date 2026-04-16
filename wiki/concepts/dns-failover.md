---
title: "DNS Failover"
type: concept
tags: [concept, dns, reliability, gcp, godaddy]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# DNS Failover

A reliability pattern where public DNS records are automatically switched to a fallback host when the primary becomes unreachable, then switched back on recovery — with no manual intervention.

## How It Is Used Here

When the home server's internet connection drops, the [[wiki/infrastructure/gcp-external-monitoring]] system detects the outage (home server unreachable from GCP) and updates GoDaddy DNS to point `camerontora.ca`, `plex.camerontora.ca`, and `seerr.camerontora.ca` at GCP's static IP (`216.239.32.21`). Visitors see the [[wiki/infrastructure/status-dashboard]] instead of a connection error. When the home server comes back, DNS reverts automatically.

## Key Properties / Mechanics

- **Detection**: GCP Cloud Run polls home server every 5 min. Failure triggers failover.
- **Execution**: GoDaddy API call updates A records for `@`, `plex`, `seerr`.
- **Fallback target**: GCP Cloud Run domain mappings serve the status dashboard for all three records.
- **Recovery**: Cloud Run monitor detects home server is back, reverts DNS.
- **DDNS sentinel**: The GoDaddy DDNS cron script checks whether `@` points to the GCP IP before running. If so, it skips the update — preventing the cron from undoing an active failover.
- **Scope**: Not all subdomains fail over — only the three most user-visible ones. Services like Radarr, Sonarr remain unreachable during an outage (acceptable, as they're internal tools).
- **Ombi excluded**: Ombi was decommissioned (2026-03-28); its static migration page lives only on the home server and intentionally doesn't fail over.

## Relationships to Other Concepts

- [[wiki/infrastructure/gcp-external-monitoring]] — detects outage and triggers failover
- [[wiki/infrastructure/dns-ssl]] — GoDaddy DDNS and the sentinel check
- [[wiki/infrastructure/status-dashboard]] — the failover destination

## External References

- GCP Cloud Run domain mappings documentation — how GCP static IP is assigned to custom domains
