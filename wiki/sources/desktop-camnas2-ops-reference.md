---
title: "Source: CAMNAS2 Server Operations Reference"
type: source
tags: [source, infra, server, docker, plex, ops]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Source: CAMNAS2 Server Operations Reference

**Raw file:** `~/Desktop/CAMNAS2_Server_Operations_Reference.md`
**Ingested:** 2026-04-17
**Type:** doc
**Last updated in source:** 2025-12-29

## Summary

Operational runbook for the CAMNAS2 home server. Covers Docker Compose basics, systemd mount dependencies, Plex config paths and backup/restore procedures, Docker disk usage and pruning, permissions, and legacy Apache2 reverse proxy commands.

The document predates Seerr migration (still references Ombi and Overseerr) so some service list content is outdated. The operational procedures (backups, Docker commands, mount dependencies) remain accurate.

## Key Facts / Data Points

- Hostname: `CAMNAS2`, OS: Ubuntu 24.04.3 LTS, primary user: `camerontora` (PUID 1000 / PGID 1000)
- Docker Compose lives at `/home/camerontora/docker-services/`
- Plex config bind-mounted at `/var/lib/plex/config`; transcode RAM disk at `/dev/shm/Transcode`
- NAS mounts (`/HOMENAS`, `/HOMENAS2`, `/CAMRAID`) enforced as Docker startup dependency via systemd unit at `/etc/systemd/system/docker.service.d/10-requires-mounts.conf`
- Recyclarr permission fix: `sudo chown -R 1000:1000 /home/camerontora/docker-services/recyclarr/config`
- Backup uses `rsync -a --delete` to `/HOMENAS/BACKUP/`; excludes Cache, Logs, Codecs, Crash Reports, Transcode from Plex backup
- Photo transcoder cache safe to clear: `rm -rf "/var/lib/plex/config/Library/Application Support/Plex Media Server/Cache/PhotoTranscoder"`
- Apache2 site configs at `/etc/apache2/sites-available/*.conf` (legacy, nginx is now primary)

## Relevance to Wiki

- [[wiki/infrastructure/home-server]] — server hostname, OS version, Docker mount dependency
- [[wiki/projects/plex-media-server]] — config path, backup/restore procedure

## Contradictions or Tensions

- Service list references Ombi (decommissioned) and Overseerr (replaced by Seerr) — outdated. Current canonical service list is in docker-compose.yaml.
- Document mentions "orphan containers" for xTeVe/Jellyfin/Emby — these are now fully removed.
- Apache2 section describes legacy reverse proxy — nginx is now the primary reverse proxy.

## Pages Updated

- [[wiki/infrastructure/home-server]] — hostname, OS version, systemd mount dependency
- [[wiki/projects/plex-media-server]] — backup/restore runbook
