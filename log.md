# camwiki Log

Append-only chronological record of all wiki operations.
Never edit or delete past entries.

Grep patterns:
- All ingests: `grep "^\#\# \[" log.md | grep "ingest"`
- All lints:   `grep "^\#\# \[" log.md | grep "lint"`
- Last 5 ops:  `grep "^\#\# \[" log.md | tail -5`

---

## [2026-04-16] bootstrap | Initial wiki scaffold

- **Pages created:** CLAUDE.md, index.md, log.md, raw/ directory tree (articles, docs, code, repos, assets), wiki/ directory tree (projects, infrastructure, integrations, decisions, business, concepts, sources)
- **Pages updated:** none
- **Notes:** Wiki initialized. 11 live git repos symlinked under raw/repos/. No sources ingested yet. Ready for first ingest.

## [2026-04-16] ingest | infrastructure repo

- **Source:** `raw/repos/infrastructure`
- **Pages created:** wiki/sources/infrastructure-repo, wiki/infrastructure/home-server, wiki/infrastructure/nginx-reverse-proxy, wiki/infrastructure/oauth2-proxy, wiki/infrastructure/gcp-external-monitoring, wiki/infrastructure/health-api, wiki/infrastructure/status-dashboard, wiki/infrastructure/gluetun-vpn, wiki/infrastructure/dns-ssl, wiki/infrastructure/storage-raid, wiki/infrastructure/docker-networks, wiki/concepts/dns-failover
- **Pages updated:** index.md
- **Notes:** First ingest. 10 infrastructure pages + 1 concept + 1 source summary. Covers full networking, SSL, monitoring, VPN, and storage topology.

## [2026-04-16] ingest | docker-services repo

- **Source:** `raw/repos/docker-services`
- **Pages created:** wiki/sources/docker-services-repo, wiki/projects/docker-services, wiki/projects/plex-media-server, wiki/projects/arr-suite, wiki/projects/transmission
- **Pages updated:** wiki/infrastructure/gluetun-vpn (3-region architecture, watcher service, memory limits, speed data), index.md
- **Notes:** Resolved contradiction with infrastructure repo — Gluetun runs 3 containers in parallel, not 1. Watchmap is a separate repo (raw/repos/watchmap) — not yet ingested.
