---
title: "Decision: External Monitoring on GCP"
type: decision
tags: [decision, adr, monitoring, gcp, infrastructure]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: External Monitoring on GCP

**Date:** 2026-01-13 (approx, based on commit history)
**Status:** `accepted`

## Context

The home server ran Uptime Kuma for service monitoring and alerting. This worked for detecting when individual services went down, but had a fundamental blind spot: Uptime Kuma runs *on* the home server. If the home internet connection drops, Uptime Kuma loses connectivity too — it can't send a Discord alert because it has no network access. The problem you most need to know about (total outage) is precisely the one the local monitor can't report.

## Options Considered

1. **Keep Uptime Kuma, accept the blind spot** — simple, no cost, but silent during internet outages.
2. **External uptime service (e.g. UptimeRobot, Better Uptime)** — handles the internet-down case but limited to HTTP checks; no access to internal metrics (CPU, RAM, disk, VPN health).
3. **GCP Cloud Run + Cloud Scheduler** — custom monitor runs outside the home network on a schedule; can check public endpoints and call the home health API for deep metrics; full control over alert logic and thresholds.

## Decision

Build a custom external monitor on GCP Cloud Run, triggered every 5 minutes by Cloud Scheduler. It checks public endpoints (`camerontora.ca`, `health.camerontora.ca`, `plex.camerontora.ca`) and sends Discord alerts on failure or recovery.

GCP was chosen over a third-party service because:
- The health API exposes internal metrics (CPU, RAM, disk, Plex library, VPN speed) that third-party services can't access
- Custom logic was needed for alert severity levels, VPN failover triggering, and DNS failover
- GCP Cloud Run has a generous free tier — this workload costs nothing

Uptime Kuma was subsequently decommissioned.

## Consequences

- Internet outages now generate Discord alerts instead of silent failures
- Enables VPN auto-failover and DNS failover as downstream features (both triggered by the GCP monitor detecting unhealthy state)
- Requires maintaining a GCP project (`cameron-tora`) and Cloud Run deployment
- GitHub Actions auto-deploys the monitor on push — no manual deploy step

## Related Pages

- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/infrastructure/health-api]]
- [[wiki/concepts/dns-failover]]

## Sources

- [[wiki/sources/infrastructure-repo]] — MONITORING.md, commit history
