---
title: "Decision: Recyclarr for Radarr/Sonarr quality profile sync"
type: decision
tags: [decision, adr, radarr, sonarr, recyclarr, media]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Recyclarr for Radarr/Sonarr quality profile sync

**Date:** 2026-04-16 (reconstructed from repo)
**Status:** `accepted`

## Context

Radarr and Sonarr require quality profiles and custom format definitions to route downloads correctly (preferred resolution, codec preferences, release group scoring). These can be configured manually in the web UI, but the configuration is:
- Not version-controlled
- Tedious to set up correctly from scratch
- Prone to drift over time as community recommendations update
- Hard to replicate after a container reset

## Options Considered

1. **Manual configuration via Radarr/Sonarr web UI** — full control, no extra service. Configuration lives in the container's SQLite database with no external representation.
2. **Export/import scripts** — back up the Radarr/Sonarr config databases manually. Restores the config but doesn't keep it aligned with community-recommended profiles.
3. **Recyclarr** — a tool that syncs quality profiles, custom formats, and scoring from the TRaSH Guides community templates into Radarr/Sonarr via API. Runs on a schedule (3:15 AM daily). Configuration is a YAML file in the repo.

## Decision

Recyclarr, running on a daily cron schedule. The TRaSH Guides templates represent community best practice for quality profiles and custom format scoring. Recyclarr applies these automatically and keeps them current as templates evolve. The YAML configuration file is version-controlled alongside docker-compose.yaml.

## Consequences

- Quality profiles stay aligned with TRaSH Guides recommendations without manual intervention.
- Configuration is declarative and version-controlled in `recyclarr/` config.
- Recyclarr overwrites manual UI changes to synced profiles — do not manually edit profiles that Recyclarr manages.
- Adds one more container and daily cron, but it is lightweight and runs quietly.

## Related Pages

- [[wiki/projects/docker-services]]

## Sources

- `raw/repos/docker-services` — docker-compose.yaml (recyclarr service), recyclarr/ config directory
