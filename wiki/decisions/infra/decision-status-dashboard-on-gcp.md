---
title: "Decision: Status Dashboard Hosted on GCP"
type: decision
tags: [decision, adr, infrastructure, gcp, status-dashboard]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Status Dashboard Hosted on GCP

**Date:** 2026-01-13 (approx)
**Status:** `accepted`

## Context

A status dashboard was needed to show service health at `status.camerontora.ca`. The obvious place to host it is the home server itself — it already runs all the other services. But a status page hosted on the home server has the same problem as local monitoring: if the home server or internet goes down, the status page goes down with it. Visitors get a connection error instead of an informative outage message.

## Options Considered

1. **Host on the home server** — simple, consistent with everything else, but unavailable during outages.
2. **Host on GCP Cloud Run** — always available from the public internet; can still display home server status even when the home server is unreachable; SSL managed by Google (CNAME, no Let's Encrypt cert needed).

## Decision

Host the status dashboard on GCP Cloud Run at `status.camerontora.ca` (CNAME to Cloud Run URL). This means the dashboard is reachable even when the home server is completely offline.

`status.camerontora.ca` is the only subdomain that is a CNAME rather than an A record — it does not go through the home nginx proxy and is not included in the Let's Encrypt `camerontora-services` certificate (Google manages its own cert).

## Consequences

- Status page is always reachable, including during home internet outages
- Doubles as the DNS failover landing page — when `camerontora.ca`, `plex.camerontora.ca`, and `seerr.camerontora.ca` fail over to GCP IPs, visitors see this dashboard
- Deployment is separate from home server (GitHub Actions → Cloud Run)
- `status.camerontora.ca` must be excluded from any SSL cert expansion commands (it's not in the Let's Encrypt cert)

## Related Pages

- [[wiki/infrastructure/status-dashboard]]
- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/concepts/dns-failover]]
- [[wiki/infrastructure/dns-ssl]]

## Sources

- [[wiki/sources/infrastructure-repo]] — STATUS-DASHBOARD.md, commit history
