---
title: "Decision: DNS Failover via GoDaddy API + Sentinel Check"
type: decision
tags: [decision, adr, dns, infrastructure, reliability]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: DNS Failover via GoDaddy API + Sentinel Check

**Date:** 2026-02-20
**Status:** `accepted`

## Context

When the home internet goes down, public visitors to `camerontora.ca` get SSL errors or timeouts. The GCP external monitor can detect the outage — but what should it do? The home server is unreachable, so it can't serve a maintenance page. Visitors need to land somewhere informative.

A secondary problem: the GoDaddy DDNS cron runs every 10 minutes and would immediately undo any DNS change made during a failover — resetting records back to the (unreachable) home IP.

## Options Considered

1. **Do nothing, accept SSL errors** — simple, no infrastructure cost, but terrible user experience.
2. **Third-party CDN failover (Cloudflare)** — migrate DNS to Cloudflare, use origin rules for failover. Requires DNS migration away from GoDaddy.
3. **GoDaddy API DNS flip + DDNS sentinel check** — flip A records to GCP static IP via the GoDaddy API (already in use for DDNS). Host the fallback on GCP Cloud Run (status dashboard, already built). Add a sentinel check to the DDNS script so it doesn't undo an active failover.

## Decision

Use the GoDaddy API (already integrated) to flip `@`, `plex`, and `seerr` A records to GCP's anycast IP (`216.239.32.21`) when the GCP monitor detects an outage. Visitors land on the status dashboard (already on GCP Cloud Run).

The DDNS sentinel: before updating DNS, the script reads the current `@` A record. If it matches a known GCP anycast IP, it exits without making changes — the cron can't undo an active failover.

**Why only these three subdomains?**
- `camerontora.ca`, `plex.camerontora.ca`, `seerr.camerontora.ca` — human-visited pages where a helpful status message beats a connection error.
- Protected services (Radarr, Sonarr, etc.) are behind OAuth — if the server is down they're just inaccessible, which is acceptable.
- `whosup.camerontora.ca` — a mobile app API endpoint; failing it over to a status HTML page would break app clients.
- `ombi.camerontora.ca` — decommissioned; static migration page intentionally stays on home server only.

## Consequences

- Public visitors see the status dashboard instead of SSL errors during outages
- DDNS cron safely coexists with active failover (sentinel check)
- GCP Cloud Run domain mappings needed for all three domains (one-time setup for cert provisioning)
- Failover and failback also triggerable manually from the status dashboard admin panel
- Adding a new subdomain to failover requires: Cloud Run domain mapping, cert provisioning, add to `DNS_RECORDS` in `config.py`

## Related Pages

- [[wiki/concepts/dns-failover]]
- [[wiki/infrastructure/dns-ssl]]
- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/infrastructure/status-dashboard]]

## Sources

- [[wiki/sources/infrastructure-repo]] — DNS-FAILOVER.md, full implementation details and test checklist
