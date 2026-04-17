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

## [2026-04-16] edit | camwiki self-documentation

- **Pages created:** wiki/projects/camwiki
- **Pages updated:** index.md
- **Notes:** The wiki now documents itself as a project — concept, architecture, linked repos, workflows, key design decisions.

## [2026-04-16] ingest | infrastructure docs/ folder — 13 markdown files

- **Source:** `raw/repos/infrastructure/docs/` (BACKLOG.md, DNS-AND-SSL.md, DNS-FAILOVER.md, MACOS-MOUNT.md, MONITORING.md, SECURITY.md, SEERR-MIGRATION.md, SSO-GUIDE.md, STATUS-DASHBOARD.md, WHOSUP.md, WHOSUP-BACKEND-REQUIREMENTS.md, WHOSUP-PROJECT-PLAN.md, SBA-BACKEND-REQUIREMENTS.md)
- **Pages created:** wiki/infrastructure/macos-remote-mount
- **Pages updated:** wiki/infrastructure/status-dashboard (services list, health check logic, admin panel, Firestore history, Netdata), wiki/infrastructure/gcp-external-monitoring (full threshold table, VPN alerting nuance), wiki/infrastructure/health-api (all endpoints, response schema, speedtest cron details), wiki/infrastructure/gluetun-vpn (orphaned transmission auto-repair), wiki/infrastructure/oauth2-proxy (per-service authorization table), wiki/projects/whosup (container name, nginx rate limits, infra status), wiki/projects/sbca (container name, infra integration), index.md
- **Notes:** SEERR-MIGRATION.md and SSO-GUIDE.md were already well-represented by the ADRs created in the prior pass. DNS-AND-SSL.md, SECURITY.md, and DNS-FAILOVER.md were largely captured in previous ingests. DNS-FAILOVER.md has a historical discrepancy — the doc still lists ombi/overseerr as failover targets but BACKLOG.md confirms these were removed; current DNS_RECORDS = ["@", "plex", "seerr"].

## [2026-04-16] edit | Gluetun VPN — corrected 3-region failover framing

- **Pages updated:** wiki/infrastructure/gluetun-vpn, wiki/projects/transmission, wiki/projects/docker-services
- **Notes:** Correction from user — the 3 Gluetun containers exist for automatic health-check failover, not manual switching. There is no preferred region; the active server is whichever is currently healthy. Also fixed duplicate source_count frontmatter bug in gluetun-vpn.md.

## [2026-04-16] ingest | all-repo docs audit — docs/ folders across 7 repos

- **Source:** `raw/repos/donormap/docs/`, `raw/repos/haymaker/docs/`, `raw/repos/rotosync/` (README), `raw/repos/sbca/` (README + app/), `raw/repos/watchmap/docs/`, `raw/repos/docker-services/docs/`, `raw/repos/whosup/docs/` (ARCHITECTURE.md, DATABASE.md, DEPLOYMENT.md)
- **Pages created:** none
- **Pages updated:** wiki/projects/donormap (data pipeline diagram, source tables, key filters, 12pm EST date window, product normalization, COA revenue rates, progress bar field naming quirk, FSA choropleth color scale, Snowflake connection details, Fire TV APK build runbook, charity logo runbook), wiki/projects/haymaker (Apple Health table schemas, sleep score formula, regularity score, workout effort score, debt forecaster methods/modes/API, new user onboarding wizard + default data init), wiki/projects/rotosync (standup AI output structure, 1:1 persistent doc, project meeting doc, Google Calendar add-on flow, countdown sync, Monday.com board IDs, Cloud Functions list), wiki/projects/sbca (full A11Y system: font sizes/touch targets/WCAG colors/4 rules, dev architecture Mac vs Ubuntu), wiki/projects/watchmap (historical markers feature, tvOS REST polling, tvOS architecture decisions: WebKit unavailable → SwiftUI+MapKit, REST not Socket.io), wiki/projects/docker-services (update command corrected for seerr, validation steps table, excluded services rationale), wiki/projects/whosup (system architecture diagram, full DB entity model with latCoarse/latExact distinction, 3-layer privacy architecture, Socket.io events table, activity types table, PM2 deployment details), index.md
- **Notes:** 7 of 11 linked repos had docs/ folders with substantive content. Budget-module-test.md and multi-user-testing-plan.md in haymaker/docs/ were historical testing plans only — no wiki value. Rotosync had no docs/ folder; content came from the repo README and source code structure. All project pages incremented source_count to 2.

## [2026-04-16] edit | decisions/ restructured with per-project subdirectories

- **Pages created:** none
- **Pages updated:** wiki/decisions/README.md, CLAUDE.md (nesting exception + filename convention)
- **Notes:** All 5 infra ADRs moved to wiki/decisions/infra/ via git mv. Cross-references updated in wiki/infrastructure/status-dashboard, wiki/infrastructure/gcp-external-monitoring, and index.md. CLAUDE.md updated with targeted nesting exception for decisions/ and clarified ADR filename convention (no decision- prefix needed inside subdirs). Going forward, project-specific ADRs live in wiki/decisions/<project>/.

## [2026-04-16] ingest | decision deep dive — camerontora.ca + docker-services

- **Source:** `raw/repos/camerontora.ca` (README, PHASE2.md, CLAUDE.md), `raw/repos/docker-services` (README, docs/VPN-MIGRATION.md, docs/UPDATING-CONTAINERS.md, docker-compose.yaml)
- **Pages created:** wiki/decisions/camerontora-ca/dedicated-oauth2-proxy, wiki/decisions/camerontora-ca/nextjs-app-router, wiki/decisions/camerontora-ca/file-based-location-cache, wiki/decisions/docker-services/transmission-gluetun-network-mode, wiki/decisions/docker-services/recyclarr-for-quality-profiles
- **Pages updated:** wiki/projects/camerontora-ca (Key Decisions section), wiki/projects/docker-services (Key Decisions upgraded with ADR links), index.md (Decisions section now grouped by project)
- **Notes:** camerontora.ca's dedicated OAuth2 Proxy is a deliberate exception to the unified SSO design — the public-by-default access model requires unauthenticated pass-through. docker-services docs were thin but VPN-MIGRATION.md captured the network_mode rationale well.

## [2026-04-16] ingest | decision deep dive — Haymaker (6 ADRs)

- **Source:** `raw/repos/haymaker` — stack-comparison.md, docs/multi-user-refactor.md, docs/apple-health-integration.md, docs/debt-payoff-forecaster.md, docker-compose.yaml, apps/api/app/deps.py, apps/api/app/routes.py, commit history (50 commits reviewed)
- **Pages created:** wiki/decisions/haymaker/fastapi-over-nextjs-api-routes, wiki/decisions/haymaker/self-hosted-not-cloud, wiki/decisions/haymaker/apple-health-push-not-pull, wiki/decisions/haymaker/multi-user-registration-flow, wiki/decisions/haymaker/pay-period-budget-model, wiki/decisions/haymaker/manual-data-priority
- **Pages updated:** wiki/projects/haymaker (Key Decisions section), index.md
- **Notes:** stack-comparison.md is a gold-standard source — explicitly documents why FastAPI over Next.js API routes, why self-hosted over Supabase/Vercel, and why OAuth2 Proxy over NextAuth. multi-user-refactor.md documents the full registration workflow. Apple Health push model driven by Apple having no server-side pull API. Pay-period budget model reflects actual income timing rather than calendar convention. Manual data priority is a deliberate UX/trust decision (commit fef4490).
