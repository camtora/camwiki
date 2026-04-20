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

## [2026-04-16] ingest | decision deep dive — SBCA + Whosup + Clarity (12 ADRs)

- **Source:** `raw/repos/sbca`, `raw/repos/whosup`, `raw/repos/clarity`, `raw/repos/deployment`
- **Pages created:** wiki/decisions/sbca/expo-over-bare-react-native, wiki/decisions/sbca/stripe-server-side-verification, wiki/decisions/sbca/environment-canada-api, wiki/decisions/sbca/expo-push-not-firebase, wiki/decisions/whosup/location-privacy-three-layers, wiki/decisions/whosup/host-approval-gate, wiki/decisions/whosup/ios-only-swiftui, wiki/decisions/whosup/postgis-deferred, wiki/decisions/clarity/android-only-platform, wiki/decisions/clarity/stripe-tap-to-pay, wiki/decisions/clarity/snowflake-as-database, wiki/decisions/clarity/twilio-sms-verification
- **Pages updated:** wiki/projects/sbca (Key Decisions section), wiki/projects/whosup (Key Decisions section), wiki/projects/clarity (Key Decisions section), index.md
- **Notes:** SBCA: Ubuntu dev + Mac for native tooling is the critical split; Stripe server-side verification documented in commit 7f4fe72; EC MSC GeoMet stations verified active 2026-03-07. Whosup: three-layer location privacy (area fuzzing at write + coarse/exact columns + distance buckets) is the core privacy design; host approval gate bundles chat + exact location into PENDING→APPROVED transition (commit 93047ae). Clarity: Snowflake used for both transactional ops and analytics (compliance legacy); Twilio webhook signature validated on every inbound SMS; monthly donations create Stripe Subscription with cancel_at 50 years forward.

## [2026-04-16] ingest | status dashboard updates — Plex banner, Ombi removal, SBA API

- **Source:** `raw/repos/infrastructure` — commits 4e2a447, 7b813ed, 4d65582, 8eb6890, 33cbb99 (status-dashboard/backend/config.py, health_checker.py, frontend/src/components/PlexStatusBanner.jsx, docs/STATUS-DASHBOARD.md)
- **Pages created:** none
- **Pages updated:** wiki/infrastructure/status-dashboard.md (services 15→14: Ombi removed, SBA API + Who's Up API moved to API category; Plex Platform Banner section added), wiki/infrastructure/nginx-reverse-proxy.md (overseerr→seerr 301 redirect noted)
- **Notes:** Ombi decommissioned and removed from config.py monitoring. SBA API added as API-category service. Plex Platform Banner polls status.plex.tv/api/v2/summary.json in parallel with health checks — amber/red banner surfaces upstream incidents so users know login/playback issues aren't the home server. jshor96@aol.com granted OAuth access (authenticated_emails.txt). dns-ssl.md already current (seerr added, ombi kept live for migration page).

## [2026-04-17] edit | Obsidian Sync setup documented

- **Pages created:** none
- **Pages updated:** wiki/projects/camwiki.md (sync method updated from Obsidian Git to Obsidian Sync; change log entry added)
- **Notes:** Obsidian installed on Ubuntu server. Obsidian Sync now transports vault to Mac and iPhone. Obsidian Git plugin on Mac decommissioned.

## [2026-04-17] edit | 10 concept pages created

- **Pages created:** wiki/concepts/webhook-patterns, wiki/concepts/stripe-payment-flows, wiki/concepts/oauth2-proxy-pattern, wiki/concepts/jwt-authentication, wiki/concepts/docker-compose-networking, wiki/concepts/websocket-realtime-events, wiki/concepts/snowflake, wiki/concepts/gcp-cloud-run, wiki/concepts/expo-react-native-build-pipeline, wiki/concepts/geospatial-postgis
- **Pages updated:** index.md (Concepts section expanded from 1 to 11 entries)
- **Notes:** Synthesised from existing wiki content — no new raw sources. Each concept documents the cross-cutting pattern and how it appears specifically in this wiki's projects.

## [2026-04-17] ingest | decision deep dives — rotosync, donormap, watchmap, camwiki (23 ADRs)

- **Source:** `raw/repos/rotosync`, `raw/repos/donormap`, `raw/repos/watchmap`, `CLAUDE.md` + `wiki/projects/camwiki.md`
- **Pages created:** wiki/decisions/rotosync/firebase-unified-backend, wiki/decisions/rotosync/daily-co-webrtc, wiki/decisions/rotosync/google-calendar-addon, wiki/decisions/rotosync/multi-meeting-type-architecture, wiki/decisions/rotosync/gemini-over-claude, wiki/decisions/rotosync/email-scoped-carryover, wiki/decisions/donormap/firestore-cache-intermediary, wiki/decisions/donormap/four-layer-domain-enforcement, wiki/decisions/donormap/fixed-bounds-map, wiki/decisions/donormap/estimated-coa-revenue, wiki/decisions/donormap/twelve-noon-time-window, wiki/decisions/donormap/firetv-pairing-code-auth, wiki/decisions/watchmap/tvos-native-swiftui, wiki/decisions/watchmap/nodejs-socket-io-backend, wiki/decisions/watchmap/single-file-frontend, wiki/decisions/watchmap/ip-api-geolocation, wiki/decisions/watchmap/proc-mount-system-stats, wiki/decisions/camwiki/pre-synthesis-over-rag, wiki/decisions/camwiki/claude-md-operating-contract, wiki/decisions/camwiki/raw-immutable, wiki/decisions/camwiki/full-path-wikilinks, wiki/decisions/camwiki/append-only-log
- **Pages updated:** wiki/projects/rotosync (Key Decisions), wiki/projects/donormap (Key Decisions), wiki/projects/watchmap (Key Decisions), wiki/projects/camwiki (Key Decisions upgraded to ADR links), index.md
- **Notes:** Rotosync: switched from Claude to Gemini (Vertex AI ADC) for AI extraction — no API key needed. DonorMap: 12pm–12pm EST window is a non-obvious business logic decision driven by face-to-face fundraising hours. Watchmap: tvOS native app was forced by WebKit unavailability on tvOS — not a preference. camwiki ADRs synthesised from CLAUDE.md itself.

## [2026-04-17] edit | Known Issues — all projects

- **Source:** `raw/repos/*` (10 repos scraped via parallel research agents)
- **Pages created:** none
- **Pages updated:** wiki/projects/camerontora-ca, wiki/projects/clarity, wiki/projects/docker-services, wiki/projects/donormap, wiki/projects/haymaker, wiki/projects/rotosync, wiki/projects/sbca, wiki/projects/watchmap, wiki/projects/whosup — each received a new `## Known Issues` section
- **Notes:** Issues grouped by severity (Critical / High / Medium / Low). Highlights: Whosup has 3 critical security/correctness issues (no token signature verification, Socket.io PM2 cluster incompatibility, capacity race condition). Clarity has Twilio signature validation disabled and a duplicate endpoint crash risk. Haymaker has a duplicate `fat_g` column that will break the next Alembic migration. Rotosync Google Meet transcript fetching is not implemented. Docker Services Gluetun memory leak and Transmission orphaning are mitigated but require operational care.

## [2026-04-17] ingest | Desktop docs — hard_drive_arrangement, CAMNAS2_Server_Operations_Reference, SSO-GUIDE

- **Source:** `~/Desktop/hard_drive_arrangement.txt`, `~/Desktop/CAMNAS2_Server_Operations_Reference.md`, `~/Desktop/SSO-GUIDE.md`
- **Pages created:** wiki/sources/desktop-hard-drive-arrangement, wiki/sources/desktop-camnas2-ops-reference, wiki/sources/desktop-sso-guide
- **Pages updated:** wiki/infrastructure/storage-raid (per-drive SMART health, CAMNAS2 drive table, open TODOs), wiki/infrastructure/home-server (hostname CAMNAS2, Ubuntu 24.04.3 LTS, Docker systemd mount dependency), wiki/projects/plex-media-server (backup/restore runbook), wiki/infrastructure/oauth2-proxy (cookie domain trick, 3 nginx patterns, 6-step checklist, troubleshooting table), index.md
- **Notes:** sde has 16 bad sectors — priority replacement candidate in HOMENAS RAID5. CAMNAS2 ops reference is dated 2025-12-29 and partially outdated (references Ombi/Overseerr). SSO guide surfaces the proxy_cookie_domain trick as the key mechanism and common failure point. Ombi credential in hard_drive_arrangement.txt deliberately omitted from wiki.

## [2026-04-17] edit | Add git history pages for all 11 repos

- **Source:** `raw/repos/*` (all 11 linked repos)
- **Pages created:** wiki/history/clarity, wiki/history/camerontora-ca, wiki/history/docker-services, wiki/history/watchmap, wiki/history/donormap, wiki/history/deployment, wiki/history/haymaker, wiki/history/infrastructure, wiki/history/rotosync, wiki/history/sbca, wiki/history/whosup
- **Pages updated:** CLAUDE.md (wiki/history/ added to directory layout; Step 4c added to ingest workflow), index.md (History section)
- **Notes:** Format is `hash  YYYY-MM-DD  subject` one line per commit, full history. Haymaker and Infrastructure are the most active repos (181 and 186 commits respectively). History pages are overwritten (not appended) on each re-ingest per Step 4c.

## [2026-04-17] edit | Obsidian Sync raw/ exclusion

- **Source:** n/a
- **Pages created:** none
- **Pages updated:** wiki/projects/camwiki.md (change log entry noting raw/ exclusion)
- **Notes:** Obsidian Sync was following symlinks in raw/repos/ and pushing node_modules from all 11 linked repos to the remote vault. Fixed by adding `raw/` to Obsidian Sync excluded folders. Remote vault will purge the raw/ content automatically.

## [2026-04-17] ingest | infrastructure repo (re-ingest: Wiki Q&A + Plex banner)

- **Source:** `raw/repos/infrastructure`
- **Pages created:** none
- **Pages updated:** wiki/infrastructure/status-dashboard (Wiki Q&A panel section, Plex banner change log), wiki/infrastructure/oauth2-proxy (jshor96 access, allowed users list), wiki/sources/infrastructure-repo (pages updated list), wiki/history/infrastructure (regenerated)
- **Notes:** Wiki Q&A panel shipped as public (no auth) — diverges from original admin-only spec. ANTHROPIC_API_KEY in Secret Manager, not plain env var. Cloud Run memory 512Mi. jshor96@aol.com added to authenticated_emails.txt.

## [2026-04-17] edit | Add scheduled-tasks concept page

- **Source:** n/a (synthesized from existing wiki pages)
- **Pages created:** wiki/concepts/scheduled-tasks
- **Pages updated:** index.md
- **Notes:** Consolidates all automated jobs into one page: 6 home server crons, 2 GCP Cloud Scheduler jobs, 3 in-process background threads, 4 app-level scheduled jobs, and frontend polling intervals.

## [2026-04-17] edit | Add Q&A log

- **Source:** n/a
- **Pages created:** wiki/qa-log.md
- **Pages updated:** CLAUDE.md (§7.2 Query Workflow — Step 3b added), index.md (Q&A Log section)
- **Notes:** All future wiki queries will be appended to wiki/qa-log.md automatically as part of the Query Workflow.

## [2026-04-17] ingest | infrastructure repo (deploy.sh refactor)

- **Source:** `raw/repos/infrastructure`
- **Pages created:** none
- **Pages updated:** wiki/infrastructure/status-dashboard (deploy.sh change log entry), wiki/history/infrastructure (regenerated)
- **Notes:** REQUIRED_SECRETS now single source of truth in deploy.sh; --set-secrets derived automatically to prevent drift.

## [2026-04-17] edit | Quartz wiki setup at wiki.camerontora.ca

- **Source:** n/a
- **Pages created:** wiki/infrastructure/quartz-wiki
- **Pages updated:** wiki/infrastructure/oauth2-proxy (wiki added to per-service table), wiki/concepts/scheduled-tasks (persistent services section), wiki/history/infrastructure (regenerated), index.md
- **Notes:** Quartz installed at /home/camerontora/quartz-wiki, content symlinked to camwiki repo root, Node 22 installed via nvm, systemd service on port 3004, OAuth2 added to nginx config.

## [2026-04-20] lint | Full wiki lint pass

- **Source:** n/a (synthesis of all wiki pages)
- **Pages created:** none
- **Pages updated:** wiki/infrastructure/quartz-wiki.md (corrected nginx proxy address, updated ignorePatterns, updated rebuild instructions)
- **Notes:** All 50 decision pages confirmed in index and linked from project pages. Main finding: quartz-wiki.md had stale host.docker.internal:3004 reference (actual: 172.21.0.1:3005). README.md files added to ignorePatterns in both quartz.config.ts and wiki docs. Known Issues for Clarity (Twilio validation) and Whosup (PM2 Socket.IO) are pre-existing code bugs — not wiki issues.
