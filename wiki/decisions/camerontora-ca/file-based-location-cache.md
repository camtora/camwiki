---
title: "Decision: File-based watch location cache (locations.json)"
type: decision
tags: [decision, adr, plex, map, storage]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: File-based watch location cache (locations.json)

**Date:** 2026-04-16 (reconstructed from repo)
**Status:** `accepted`

## Context

The Plex watch history map needs geolocated IP data from Tautulli. Tautulli stores IP addresses but not coordinates. Each IP must be resolved via ip-api.com. The question was how to persist this resolved data and share it between camerontora.ca and the Watchmap backend.

## Options Considered

1. **Database** — PostgreSQL or SQLite table of `ip → {lat, lng, label}`. Queryable, relational. But: adds a DB dependency to a project that has none; requires schema management.
2. **File (`locations.json`)** — A JSON array written by the backfill script and by the Watchmap backend. Read by camerontora.ca's `/api/tautulli` route. Simple, no dependencies.
3. **Tautulli API on every request** — Geolocate IPs at request time. Slow; hits ip-api.com rate limits; no caching.

## Decision

`app/data/locations.json` — a flat JSON file written by two producers (Watchmap backend and backfill script) and read by the camerontora.ca API route. The file is:
- **Gitignored** for privacy (contains IP-derived coordinates)
- **Bind-mounted** into the Watchmap container so Watchmap can write it
- **Hourly cron** (`0 * * * *`) runs `backfill-locations.ts` to add any new Tautulli history

## Consequences

- No database dependency in camerontora.ca — the entire stack is stateless beyond this one file.
- Cross-repo dependency: Watchmap writes the file; camerontora.ca reads it. If Watchmap is stopped, the file still exists and the map still works from historical data.
- No deduplication guarantees — but the map rendering handles duplicates visually.
- If the file is lost, backfill script can regenerate it from Tautulli history.

## Related Pages

- [[wiki/projects/camerontora-ca]]
- [[wiki/projects/watchmap]]

## Sources

- `raw/repos/camerontora.ca` — CLAUDE.md (data flow), scripts/backfill-locations.ts, app/app/api/tautulli/route.ts
