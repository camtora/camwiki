---
title: "Decision: ip-api.com for geolocation with 24-hour TTL cache"
type: decision
tags: [decision, adr, geolocation, ip-api, caching]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: ip-api.com for geolocation with 24-hour TTL cache

**Date:** 2026-04-17 (reconstructed from initial commit 5b2c94f)
**Status:** `accepted`

## Context

Watchmap plots active Plex stream locations on a world map. Each stream has a viewer IP address that must be resolved to geographic coordinates. The geolocation service must be free, require no setup, and handle repeated lookups for the same IPs efficiently.

## Options Considered

1. **MaxMind GeoLite2** — local database, no API quota, accurate. Requires regular database downloads and maintenance.
2. **ip-api.com** — free, no API key, HTTP request per lookup, rate-limited at ~45 req/min on free tier.
3. **ip-api.com + in-memory cache** — same as above but cache results for 24 hours; most Plex viewers have stable IPs.

## Decision

ip-api.com with a 24-hour TTL in-memory cache (`CACHE_TTL_MS = 1000 * 60 * 60 * 24`).

- **No API key required** — zero-friction deployment; no secret management.
- **24-hour cache** — most home viewers have stable IPs; caching eliminates repeated lookups and stays well within api-api.com's rate limit.
- **LAN/private IP detection** — `isPrivateIp()` detects RFC1918 ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) and maps them to a configurable home location (`HOME_LAT`, `HOME_LON`, `HOME_LABEL` env vars in docker-compose.yml).

## Consequences

- City-level accuracy (~5–50km radius depending on ISP). Sufficient for placing a marker on a world map.
- Cache is in-memory — cleared on container restart. First requests after restart trigger fresh API calls.
- Free tier rate limit (45 req/min) is not a concern: Plex typically has <10 concurrent streams, cached for 24 hours.

## Related Pages

- [[wiki/projects/watchmap]]

## Sources

- `raw/repos/watchmap` — backend/server.js (lines 222–272), docker-compose.yml
