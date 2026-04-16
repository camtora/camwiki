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

## [2026-04-16] ingest | camerontora.ca repo

- **Source:** `raw/repos/camerontora.ca`
- **Pages created:** wiki/sources/camerontora-ca-repo, wiki/projects/camerontora-ca
- **Pages updated:** wiki/infrastructure/oauth2-proxy (added second instance), index.md
- **Notes:** Discovered cross-repo dependency — Watchmap writes data/locations.json, camerontora.ca reads it. camerontora.ca has its own OAuth2 Proxy instance separate from the shared infrastructure one.

## [2026-04-16] ingest | watchmap, haymaker, whosup, rotosync, donormap, sbca, clarity, deployment repos

- **Source:** `raw/repos/watchmap`, `raw/repos/haymaker`, `raw/repos/whosup`, `raw/repos/rotosync`, `raw/repos/donormap`, `raw/repos/sbca`, `raw/repos/clarity`, `raw/repos/deployment`
- **Pages created:** wiki/projects/watchmap, wiki/projects/haymaker, wiki/projects/whosup, wiki/projects/rotosync, wiki/projects/donormap, wiki/projects/sbca, wiki/projects/clarity + 8 source summaries
- **Pages updated:** index.md
- **Notes:** Confirmed deployment repo is backend for Clarity (not DonorMap) — different Snowflake database (PHOENIX_APP_DEV vs Fivetran), different tech (FastAPI vs Firebase Functions). All 11 linked repos now ingested. Haymaker documented in depth (largest project). Rotosync switched from Claude to Gemini/Vertex AI for AI summaries.

## [2026-04-16] ingest | infrastructure decisions retro (5 ADRs)

- **Source:** `raw/repos/infrastructure`
- **Pages created:** wiki/decisions/decision-gcp-external-monitoring, wiki/decisions/decision-status-dashboard-on-gcp, wiki/decisions/decision-unified-oauth2-proxy, wiki/decisions/decision-dns-failover-approach, wiki/decisions/decision-overseerr-to-seerr
- **Pages updated:** index.md
- **Notes:** Retrospective pass on the infrastructure repo to capture architectural decisions as ADRs. 5 decisions documented covering monitoring, status dashboard hosting, SSO auth strategy, DNS failover design, and the Overseerr→Seerr phased migration (Phases 2–3 still pending).

## [2026-04-16] edit | Gluetun VPN — corrected 3-region failover framing

- **Pages updated:** wiki/infrastructure/gluetun-vpn, wiki/projects/transmission, wiki/projects/docker-services
- **Notes:** Correction from user — the 3 Gluetun containers exist for automatic health-check failover, not manual switching. There is no preferred region; the active server is whichever is currently healthy. Also fixed duplicate source_count frontmatter bug in gluetun-vpn.md.
