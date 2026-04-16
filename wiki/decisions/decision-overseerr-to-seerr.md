---
title: "Decision: Migrate Overseerr to Seerr"
type: decision
tags: [decision, adr, media, plex, migration]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Migrate Overseerr to Seerr

**Date:** 2026-03-28 (Phase 1 complete)
**Status:** `accepted` (Phase 1 done; Phase 2 container swap pending)

## Context

Overseerr is the media request tool used at `overseerr.camerontora.ca`. Seerr is its designated successor — a maintained fork that merges Overseerr and Jellyseerr. Overseerr's original development has stalled. Staying on Overseerr means accumulating technical debt against an unmaintained project.

## Options Considered

1. **Stay on Overseerr** — no work required, but no future updates or security fixes.
2. **Big-bang migration** — swap the container in one step, update all DNS/nginx/certs at once. Fast but risky; if the Seerr container has issues, the media request service is down while debugging.
3. **Phased migration** — add `seerr.camerontora.ca` pointing to the same port (still running Overseerr), redirect `overseerr.camerontora.ca` → `seerr.camerontora.ca`, then swap the container separately once users are on the new URL.

## Decision

Phased migration (option 3):

- **Phase 1 (done, 2026-03-28):** Add `seerr.camerontora.ca`, 301-redirect from `overseerr`, update DDNS + SSL cert + status dashboard monitoring.
- **Phase 2 (pending):** Swap the container image from Overseerr to Seerr, fix permissions (Seerr runs as UID 1000), add `init: true`. Seerr auto-migrates the database.
- **Phase 3 (pending, after Phase 2):** Remove the `overseerr.camerontora.ca` redirect block, remove from DDNS and SSL cert.

The phased approach means the URL transition happens independently of the container swap — users move to the new URL while still hitting the old software. The container swap can be timed separately with a quick rollback path.

## Consequences

- `overseerr.camerontora.ca` transparently redirects to `seerr.camerontora.ca`; old bookmarks/links continue to work
- Phase 2 requires: config backup, `chown -R 1000:1000` on the config dir, image change in docker-services, health-api service name update
- Phase 3 shrinks the SSL cert by one domain (safe to do at any time)
- Ombi (previous request tool, replaced by Overseerr) was fully decommissioned at the same time; static migration page remains at `ombi.camerontora.ca` on the home server but is excluded from DNS failover

## Related Pages

- [[wiki/projects/docker-services]]
- [[wiki/infrastructure/dns-ssl]]

## Sources

- [[wiki/sources/infrastructure-repo]] — SEERR-MIGRATION.md, commit history
