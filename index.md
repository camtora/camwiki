# camwiki Index

Master catalog of all wiki pages. Updated by Claude on every ingest and whenever
a new page is created. Read this first when answering queries.

Last updated: 2026-04-17 (desktop docs ingest + Known Issues pass)

---

## Projects

Home-server projects — software stacks, services, personal tools.

- [[wiki/projects/arr-suite|Arr Suite]] — Radarr/Sonarr/Jackett/FlareSolverr/Recyclarr media automation pipeline. `1 source`
- [[wiki/projects/camwiki|camwiki]] — This wiki; LLM-maintained engineering knowledge base using Claude + Obsidian. `1 source`
- [[wiki/projects/camerontora-ca|camerontora.ca]] — Personal landing page and home dashboard; Plex watch history map, service tiles, Google SSO. `1 source`
- [[wiki/projects/clarity|Clarity]] — Android door-to-door fundraising app for GlobalFaces Direct; Stripe tap-to-pay, Twilio SMS, Snowflake. `2 sources`
- [[wiki/projects/donormap|DonorMap]] — Real-time donor viz map for GlobalFaces Direct; Snowflake pipeline, FSA choropleth, Fire TV + APK build runbook. `2 sources`
- [[wiki/projects/haymaker|Haymaker]] — Personal health/habit/finance tracker; FastAPI + Next.js + PostgreSQL; Apple Health push model, sleep/effort score formulas, debt forecaster. `2 sources`
- [[wiki/projects/rotosync|Rotosync]] — Standup app; Daily.co WebRTC, Gemini AI summaries, Monday.com boards, Calendar add-on, 1:1 + Project meeting docs. `2 sources`
- [[wiki/projects/sbca|SBCA]] — Cottage association mobile app (Expo); Stripe dues, lake health, A11Y system (18pt body min, WCAG AA colors, 3-channel status). `2 sources`
- [[wiki/projects/watchmap|Watchmap]] — Real-time Plex stream map; Leaflet + Socket.IO; tvOS native SwiftUI (WebKit unavailable on tvOS). `2 sources`
- [[wiki/projects/whosup|Who's Up]] — Map-first social app; PostGIS, 3-layer location privacy, Socket.io events, 12 activity types, PM2 cluster. `2 sources`
- [[wiki/projects/docker-services|Docker Services]] — Home media server Docker Compose stack; all services, 3-region Gluetun VPN, update runbook. `2 sources`
- [[wiki/projects/plex-media-server|Plex Media Server]] — Plex + Tautulli, streaming from HOMENAS with RAM-disk transcoding. `1 source`
- [[wiki/projects/transmission|Transmission]] — Torrent client running inside Gluetun VPN network namespace. `1 source`

---

## Infrastructure

Hosts, networks, storage, operating systems, container runtimes.

- [[wiki/infrastructure/home-server|Home Server]] — Primary Ubuntu home server (CAMNAS2, Ubuntu 24.04.3 LTS), port forwarding, UFW, RealVNC, Docker mount dependency. `2 sources`
- [[wiki/infrastructure/nginx-reverse-proxy|Nginx Reverse Proxy]] — SSL termination and subdomain routing for all *.camerontora.ca services. `1 source`
- [[wiki/infrastructure/oauth2-proxy|OAuth2 Proxy]] — Two instances: shared infrastructure SSO + camerontora.ca-specific auth; nginx patterns, 6-step checklist, troubleshooting. `3 sources`
- [[wiki/infrastructure/dns-ssl|DNS and SSL]] — GoDaddy DDNS (cron/10min) and Let's Encrypt shared cert (19 subdomains). `1 source`
- [[wiki/infrastructure/docker-networks|Docker Networks]] — Docker Compose network topology and UFW subnet rules. `1 source`
- [[wiki/infrastructure/gcp-external-monitoring|GCP External Monitoring]] — Cloud Run monitor checks home server every 5 min, Discord alerts, DNS failover, VPN failover; full thresholds. `2 sources`
- [[wiki/infrastructure/gluetun-vpn|Gluetun VPN]] — 3-region PIA WireGuard VPN (Toronto/Montreal/Vancouver), auto-failover, orphan auto-repair. `2 sources`
- [[wiki/infrastructure/health-api|Health API]] — Flask container exposing CPU/RAM/disk/Plex/SMART metrics; all endpoints including admin. `2 sources`
- [[wiki/infrastructure/macos-remote-mount|macOS Remote Mount]] — SSHFS mount of home server storage on MacBook (home + remote). `1 source`
- [[wiki/infrastructure/status-dashboard|Status Dashboard]] — GCP-hosted dashboard; 15 services monitored; admin panel (reboot, VPN, restart); Firestore history. `2 sources`
- [[wiki/infrastructure/storage-raid|Storage and RAID]] — HOMENAS (10-drive software RAID5, sde has 16 bad sectors) and CAMRAID (hardware RAID5); per-drive SMART health. `2 sources`

---

## Integrations

External APIs, third-party services, data flows connecting the home server to
the outside world.

_No entries yet._

---

## Decisions

Architecture decision records and significant choices.

**camerontora.ca**
- [[wiki/decisions/camerontora-ca/dedicated-oauth2-proxy|Dedicated OAuth2 Proxy]] — Own proxy (port 4182) vs shared infra; site is public-by-default. `1 source`
- [[wiki/decisions/camerontora-ca/nextjs-app-router|Next.js App Router]] — SSR header reading for auth; API routes eliminate separate backend. `1 source`
- [[wiki/decisions/camerontora-ca/file-based-location-cache|File-based Location Cache]] — locations.json vs DB; shared via bind mount with Watchmap. `1 source`

**docker-services**
- [[wiki/decisions/docker-services/transmission-gluetun-network-mode|Transmission in Gluetun Namespace]] — Strictest VPN kill-switch via network_mode; no leak possible. `1 source`
- [[wiki/decisions/docker-services/recyclarr-for-quality-profiles|Recyclarr for Quality Profiles]] — TRaSH Guides sync into Radarr/Sonarr; config version-controlled. `1 source`

**haymaker**
- [[wiki/decisions/haymaker/fastapi-over-nextjs-api-routes|FastAPI over Next.js API Routes]] — Compute-heavy logic warrants Python; budget forecasting, debt simulation, macro math. `1 source`
- [[wiki/decisions/haymaker/self-hosted-not-cloud|Self-hosted, Not Cloud]] — Health and financial data off third-party cloud intentionally; PostgreSQL + MinIO. `1 source`
- [[wiki/decisions/haymaker/apple-health-push-not-pull|Apple Health Push Model]] — Health Auto Export webhook; Apple has no server-side pull API. `1 source`
- [[wiki/decisions/haymaker/multi-user-registration-flow|Multi-user Registration Flow]] — OAuth2 allowlist + Discord approval; no self-serve registration. `1 source`
- [[wiki/decisions/haymaker/pay-period-budget-model|Pay-period Budget Model]] — Pay periods not calendar months; matches actual income timing. `1 source`
- [[wiki/decisions/haymaker/manual-data-priority|Manual Data Priority]] — Manual entry authoritative; auto-sync supplements analytics only. `1 source`

**camwiki**
- [[wiki/decisions/camwiki/pre-synthesis-over-rag|Pre-synthesis over RAG]] — Wiki pages maintained over time; compounding value vs. re-deriving at query time. `0 sources`
- [[wiki/decisions/camwiki/claude-md-operating-contract|CLAUDE.md as Operating Contract]] — Auto-read by Claude Code; all rules in one self-activating file. `0 sources`
- [[wiki/decisions/camwiki/raw-immutable|raw/ is Immutable]] — Source documents never modified; enables auditability and clean re-ingest. `0 sources`
- [[wiki/decisions/camwiki/full-path-wikilinks|Full-path Wiki-links]] — Collision-safe and refactoring-safe as wiki scales to org-level. `0 sources`
- [[wiki/decisions/camwiki/append-only-log|Append-only log.md]] — Immutable audit trail of every operation; grep-searchable. `0 sources`

**donormap**
- [[wiki/decisions/donormap/firestore-cache-intermediary|Firestore Cache Intermediary]] — 15-min Snowflake refresh; browser subscribes via real-time listeners. `1 source`
- [[wiki/decisions/donormap/four-layer-domain-enforcement|Four-layer Domain Enforcement]] — OAuth + frontend + Cloud Function + Firestore rules; defense-in-depth. `1 source`
- [[wiki/decisions/donormap/fixed-bounds-map|Fixed-bounds Map]] — No zoom or pan; broadcast display; consistent view across TVs. `1 source`
- [[wiki/decisions/donormap/estimated-coa-revenue|Estimated COA Revenue]] — Fixed rates per product type; labelled as estimate. `1 source`
- [[wiki/decisions/donormap/twelve-noon-time-window|12pm–12pm Time Window]] — Midday-to-midday captures full business day; eliminates timezone edge cases. `1 source`
- [[wiki/decisions/donormap/firetv-pairing-code-auth|Fire TV Pairing Code Auth]] — 6-digit code + custom Firebase token; OAuth impossible on sideloaded APK. `1 source`

**rotosync**
- [[wiki/decisions/rotosync/firebase-unified-backend|Firebase as Unified Backend]] — Firestore + Cloud Functions + Auth + Hosting; real-time multi-user platform. `1 source`
- [[wiki/decisions/rotosync/daily-co-webrtc|Daily.co for WebRTC and Transcription]] — Built-in live transcription + active speaker events. `1 source`
- [[wiki/decisions/rotosync/google-calendar-addon|Google Calendar Add-on]] — Native conferencing provider; meeting metadata from calendar. `1 source`
- [[wiki/decisions/rotosync/multi-meeting-type-architecture|Multi-meeting-type Architecture]] — Title-pattern detection; default-to-1:1; per-type carryover visibility. `1 source`
- [[wiki/decisions/rotosync/gemini-over-claude|Gemini over Claude for AI Extraction]] — Vertex AI ADC; no API key secret needed. `1 source`
- [[wiki/decisions/rotosync/email-scoped-carryover|Email-scoped Carryover Items]] — Email matching with alias normalisation; 1:1 isolation. `1 source`

**watchmap**
- [[wiki/decisions/watchmap/tvos-native-swiftui|tvOS Native SwiftUI]] — WebKit unavailable on tvOS; native MapKit with REST polling. `1 source`
- [[wiki/decisions/watchmap/nodejs-socket-io-backend|Node.js + Socket.IO Backend]] — Real-time push to browsers; minimal Alpine container. `1 source`
- [[wiki/decisions/watchmap/single-file-frontend|Single-file Frontend]] — No build step; volume-mounted; zero frontend dependencies. `1 source`
- [[wiki/decisions/watchmap/ip-api-geolocation|ip-api.com Geolocation]] — Free, no API key; 24-hour cache; LAN IP override. `1 source`
- [[wiki/decisions/watchmap/proc-mount-system-stats|Host /proc Mount for System Stats]] — Read-only proc mount; accurate real-time CPU/RAM. `1 source`

**sbca**
- [[wiki/decisions/sbca/expo-over-bare-react-native|Expo over Bare React Native]] — Managed workflow for Ubuntu-based dev loop; expo prebuild + Mac native tooling. `1 source`
- [[wiki/decisions/sbca/stripe-server-side-verification|Stripe Server-side Payment Verification]] — Backend re-fetches PaymentIntent and validates metadata binding against member ID. `1 source`
- [[wiki/decisions/sbca/environment-canada-api|Environment Canada MSC GeoMet API]] — Free hydrometric API; stations 02EB018 (Beaumaris) and 02EB008 (Baysville). `1 source`
- [[wiki/decisions/sbca/expo-push-not-firebase|Expo Push over Firebase]] — Single Expo endpoint; no Firebase project or native SDK required. `1 source`

**whosup**
- [[wiki/decisions/whosup/location-privacy-three-layers|Three-layer Location Privacy]] — Area fuzzing at write time + coarse/exact split + distance buckets. `1 source`
- [[wiki/decisions/whosup/host-approval-gate|Host Approval Gate]] — PENDING→APPROVED/DENIED; chat and exact location bundled into approval. `1 source`
- [[wiki/decisions/whosup/ios-only-swiftui|iOS-only SwiftUI]] — MapKit, Sign in with Apple (App Store requirement for social apps), iOS 17+. `1 source`
- [[wiki/decisions/whosup/postgis-deferred|PostGIS Deferred]] — PostGIS installed but Haversine at app level for now; clear migration path. `1 source`

**clarity**
- [[wiki/decisions/clarity/android-only-platform|Android-only Platform]] — Stripe Tap to Pay on Android first; enterprise tablets cheaper; NFC required. `1 source`
- [[wiki/decisions/clarity/stripe-tap-to-pay|Stripe Tap to Pay on Android]] — Device NFC as card reader; monthly donations create Stripe Subscription. `1 source`
- [[wiki/decisions/clarity/snowflake-as-database|Snowflake as Database]] — Compliance legacy; immutable EVENT_LOG; signatures in internal stage. `1 source`
- [[wiki/decisions/clarity/twilio-sms-verification|Twilio SMS Verification]] — Mandatory YES/NO consent; Twilio signature validation; SMS events logged for audit. `1 source`

**infra**
- [[wiki/decisions/infra/decision-gcp-external-monitoring|GCP External Monitoring]] — Why build a custom GCP Cloud Run monitor instead of keeping Uptime Kuma or using a third-party service. `1 source`
- [[wiki/decisions/infra/decision-status-dashboard-on-gcp|Status Dashboard on GCP]] — Why host the status dashboard on GCP Cloud Run rather than the home server. `1 source`
- [[wiki/decisions/infra/decision-unified-oauth2-proxy|Unified OAuth2 Proxy]] — Why deploy a single shared OAuth2 Proxy for SSO across all protected services. `1 source`
- [[wiki/decisions/infra/decision-dns-failover-approach|DNS Failover Approach]] — GoDaddy API flip + DDNS sentinel check; why only 3 subdomains fail over. `1 source`
- [[wiki/decisions/infra/decision-overseerr-to-seerr|Overseerr → Seerr Migration]] — Phased migration strategy; Phase 1 done, Phases 2–3 pending. `1 source`

---

## Business

Contracts, clients, vendors, metric definitions. Populated when the wiki scales
to an org-level knowledge base.

_No entries yet._

---

## Concepts

Technology reference knowledge, principles, and cross-cutting ideas.

- [[wiki/concepts/dns-failover|DNS Failover]] — Pattern for auto-switching DNS to a fallback host on outage and reverting on recovery. `1 source`
- [[wiki/concepts/docker-compose-networking|Docker Compose Networking]] — Bridge networks, service-name DNS, and network_mode:service: for VPN namespace sharing. `0 sources`
- [[wiki/concepts/expo-react-native-build-pipeline|Expo / React Native Build Pipeline]] — Ubuntu+Mac split dev, expo prebuild, Metro subdomain, xcworkspace gotcha. `0 sources`
- [[wiki/concepts/gcp-cloud-run|GCP Cloud Run]] — Serverless containers; used for status dashboard and external monitor; scales to zero. `0 sources`
- [[wiki/concepts/geospatial-postgis|Geospatial / PostGIS]] — PostGIS spatial indexes, ST_DWithin, Haversine vs DB-layer queries, location fuzzing. `0 sources`
- [[wiki/concepts/jwt-authentication|JWT Authentication]] — Stateless token auth; access+refresh pattern (SBCA) vs single long-lived token (Whosup). `0 sources`
- [[wiki/concepts/oauth2-proxy-pattern|OAuth2 Proxy Pattern]] — Nginx sidecar auth; X-Forwarded-Email; 401=Up; email allowlist; webhook bypass. `0 sources`
- [[wiki/concepts/snowflake|Snowflake]] — OLAP warehouse used as OLTP (Clarity); immutable EVENT_LOG; internal stage for signatures. `0 sources`
- [[wiki/concepts/stripe-payment-flows|Stripe Payment Flows]] — Terminal tap-to-pay (Clarity) vs CardField+server-side verification (SBCA). `0 sources`
- [[wiki/concepts/websocket-realtime-events|WebSocket / Real-time Events]] — Socket.io rooms, event patterns (Whosup), PM2 sticky sessions. `0 sources`
- [[wiki/concepts/webhook-patterns|Webhook Patterns]] — Signature validation, idempotency via event_id, public endpoint requirement. `0 sources`

---

## Sources

One entry per raw source ingested.

- [[wiki/sources/infrastructure-repo|infrastructure repo]] — Full infrastructure repo: nginx, OAuth2, DDNS, SSL, GCP monitoring, VPN, RAID. `1 source`
- [[wiki/sources/camerontora-ca-repo|camerontora.ca repo]] — Personal landing page: Next.js, Plex map, service tiles, dual OAuth2 setup. `1 source`
- [[wiki/sources/clarity-repo|clarity repo]] — GlobalFaces Android fundraising app; Stripe tap-to-pay, Twilio SMS. `1 source`
- [[wiki/sources/deployment-repo|deployment repo]] — FastAPI backend for Clarity; Stripe, Twilio, Snowflake donor data. `1 source`
- [[wiki/sources/donormap-repo|donormap repo]] — GlobalFaces donor map; Firebase + Snowflake + Fire TV. `1 source`
- [[wiki/sources/haymaker-repo|haymaker repo]] — Health/habit/finance tracker; full architecture + stack decisions. `1 source`
- [[wiki/sources/rotosync-repo|rotosync repo]] — Standup app; Firebase + Daily.co + Gemini AI. `1 source`
- [[wiki/sources/sbca-repo|sbca repo]] — Cottage association app; Expo + Fastify + Stripe + Environment Canada. `1 source`
- [[wiki/sources/watchmap-repo|watchmap repo]] — Plex stream map; Node.js + Socket.IO + tvOS app. `1 source`
- [[wiki/sources/whosup-repo|whosup repo]] — Social activity app; Node.js + PostGIS + SwiftUI. `1 source`
- [[wiki/sources/docker-services-repo|docker-services repo]] — Home media stack: Plex, *arr suite, Transmission, multi-region Gluetun VPN. `1 source`
- [[wiki/sources/desktop-hard-drive-arrangement|Hard Drive Arrangement]] — Per-drive UUID and SMART health snapshot; sde has 16 bad sectors; CAMNAS2 drive labels; open maintenance TODOs. `1 source`
- [[wiki/sources/desktop-camnas2-ops-reference|CAMNAS2 Ops Reference]] — Server hostname/OS, Docker systemd mount dependency, Plex backup/restore runbook (2025-12-29). `1 source`
- [[wiki/sources/desktop-sso-guide|SSO Guide]] — Nginx config patterns for protected/public/hybrid services; 6-step checklist; troubleshooting. `1 source`
