---
title: "Plex Media Server"
type: project
tags: [project, media, plex, tautulli]
created: 2026-04-16
updated: 2026-04-17
source_count: 2
---

# Plex Media Server

Home media server streaming movies and TV shows from HOMENAS. Paired with Tautulli for analytics and monitoring.

## Status

Current status: `active`
Last meaningful change: 2026-01-13

## Architecture

| Component | Port | Role |
|-----------|------|------|
| Plex | 32400 | Media server (public, no OAuth2) |
| Tautulli | 8181 | Analytics, API source for Watchmap |

Plex is publicly accessible at `plex.camerontora.ca` without authentication (Plex has its own account system). Tautulli is protected by OAuth2.

## Infrastructure

- Config persisted at `/var/lib/plex/config` on the host (bind-mounted — safe across container recreations)
- Media library on `/HOMENAS`
- Transcoding uses `/dev/shm` (RAM disk) to avoid disk I/O during transcode sessions
- Plex availability monitored by [[wiki/infrastructure/gcp-external-monitoring]] every 5 min

## Integrations

- **Tautulli API** — consumed by [[wiki/projects/watchmap]] (stream location data) and the camerontora.ca status banner
- **Radarr/Sonarr** — add completed downloads to Plex library automatically (see [[wiki/projects/arr-suite]])
- **Health API** — Plex library availability checked via Plex API and exposed at `health.camerontora.ca`

## Backup & Restore

### Backup

```bash
DATE=$(date +%F)

# Docker Compose configs
rsync -a --delete \
  /home/camerontora/docker-services/ \
  /HOMENAS/BACKUP/docker-services-$DATE/

# Plex config (exclude transient folders)
rsync -a --delete \
  --exclude='*/Crash Reports/*' \
  --exclude='*/Logs/*' \
  --exclude='*/Codecs/*' \
  --exclude='*/Cache/*' \
  --exclude='*/Transcode/*' \
  /var/lib/plex/config/ \
  /HOMENAS/BACKUP/plex-config-$DATE/
```

### Restore Library (after container recreation)

```bash
docker stop plex
rm -rf /var/lib/plex/config/Library
cp -r /HOMENAS/BACKUP/plex-config-YYYY-MM-DD/Library /var/lib/plex/config/
sudo chown -R 1000:1000 /var/lib/plex/config
docker start plex
```

### Clear Photo Transcoder Cache (safe)

```bash
rm -rf "/var/lib/plex/config/Library/Application Support/Plex Media Server/Cache/PhotoTranscoder"
```

## Key Decisions

- Config bind-mounted to host (`/var/lib/plex/config`) rather than a Docker volume — makes backups and migrations straightforward
- RAM disk transcoding (`/dev/shm`) keeps transcode I/O off the NAS

## Change Log

- 2026-01-13: Plex monitoring added to GCP external monitor

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/docker-services-repo]] — container config, storage mounts, transcoding setup
- [[wiki/sources/desktop-camnas2-ops-reference]] — backup/restore runbook, photo transcoder cache clear
