# camwiki Index

Master catalog of all wiki pages. Updated by Claude on every ingest and whenever
a new page is created. Read this first when answering queries.

Last updated: 2026-04-16 (docs audit pass)

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

- [[wiki/infrastructure/home-server|Home Server]] — Primary Ubuntu home server, port forwarding, UFW, RealVNC, secrets. `1 source`
- [[wiki/infrastructure/nginx-reverse-proxy|Nginx Reverse Proxy]] — SSL termination and subdomain routing for all *.camerontora.ca services. `1 source`
- [[wiki/infrastructure/oauth2-proxy|OAuth2 Proxy]] — Two instances: shared infrastructure SSO + camerontora.ca-specific auth; per-service auth table. `2 sources`
- [[wiki/infrastructure/dns-ssl|DNS and SSL]] — GoDaddy DDNS (cron/10min) and Let's Encrypt shared cert (19 subdomains). `1 source`
- [[wiki/infrastructure/docker-networks|Docker Networks]] — Docker Compose network topology and UFW subnet rules. `1 source`
- [[wiki/infrastructure/gcp-external-monitoring|GCP External Monitoring]] — Cloud Run monitor checks home server every 5 min, Discord alerts, DNS failover, VPN failover; full thresholds. `2 sources`
- [[wiki/infrastructure/gluetun-vpn|Gluetun VPN]] — 3-region PIA WireGuard VPN (Toronto/Montreal/Vancouver), auto-failover, orphan auto-repair. `2 sources`
- [[wiki/infrastructure/health-api|Health API]] — Flask container exposing CPU/RAM/disk/Plex/SMART metrics; all endpoints including admin. `2 sources`
- [[wiki/infrastructure/macos-remote-mount|macOS Remote Mount]] — SSHFS mount of home server storage on MacBook (home + remote). `1 source`
- [[wiki/infrastructure/status-dashboard|Status Dashboard]] — GCP-hosted dashboard; 15 services monitored; admin panel (reboot, VPN, restart); Firestore history. `2 sources`
- [[wiki/infrastructure/storage-raid|Storage and RAID]] — HOMENAS (8-drive software RAID5) and CAMRAID (hardware RAID5) with SMART monitoring. `1 source`

---

## Integrations

External APIs, third-party services, data flows connecting the home server to
the outside world.

_No entries yet._

---

## Decisions

Architecture decision records and significant choices.

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
