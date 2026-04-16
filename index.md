# camwiki Index

Master catalog of all wiki pages. Updated by Claude on every ingest and whenever
a new page is created. Read this first when answering queries.

Last updated: 2026-04-16

---

## Projects

Home-server projects — software stacks, services, personal tools.

- [[wiki/projects/arr-suite|Arr Suite]] — Radarr/Sonarr/Jackett/FlareSolverr/Recyclarr media automation pipeline. `1 source`
- [[wiki/projects/camerontora-ca|camerontora.ca]] — Personal landing page and home dashboard; Plex watch history map, service tiles, Google SSO. `1 source`
- [[wiki/projects/clarity|Clarity]] — Android door-to-door fundraising app for GlobalFaces Direct; Stripe tap-to-pay, Twilio SMS, Snowflake. `2 sources`
- [[wiki/projects/donormap|DonorMap]] — Real-time donor viz map for GlobalFaces Direct; Firebase + Snowflake + Fire TV office displays. `1 source`
- [[wiki/projects/haymaker|Haymaker]] — Personal health/habit/finance tracker; FastAPI + Next.js + PostgreSQL; Withings + Apple Health integrations. `1 source`
- [[wiki/projects/rotosync|Rotosync]] — Standup facilitation app; Daily.co WebRTC, live transcription, Gemini AI summaries, Monday.com. `1 source`
- [[wiki/projects/sbca|SBCA]] — Cottage association mobile app (Expo); Stripe dues, lake health dashboard, community board. `1 source`
- [[wiki/projects/watchmap|Watchmap]] — Real-time Plex stream map; Node.js + Socket.IO + Leaflet; tvOS app. `1 source`
- [[wiki/projects/whosup|Who's Up]] — Map-first social activity app; Node.js + PostGIS + SwiftUI iOS; Muskoka/Toronto/Santa Teresa. `1 source`
- [[wiki/projects/docker-services|Docker Services]] — Home media server Docker Compose stack, all services and VPN setup. `1 source`
- [[wiki/projects/plex-media-server|Plex Media Server]] — Plex + Tautulli, streaming from HOMENAS with RAM-disk transcoding. `1 source`
- [[wiki/projects/transmission|Transmission]] — Torrent client running inside Gluetun VPN network namespace. `1 source`

---

## Infrastructure

Hosts, networks, storage, operating systems, container runtimes.

- [[wiki/infrastructure/home-server|Home Server]] — Primary Ubuntu home server, port forwarding, UFW, RealVNC, secrets. `1 source`
- [[wiki/infrastructure/nginx-reverse-proxy|Nginx Reverse Proxy]] — SSL termination and subdomain routing for all *.camerontora.ca services. `1 source`
- [[wiki/infrastructure/oauth2-proxy|OAuth2 Proxy]] — Two instances: shared infrastructure SSO + camerontora.ca-specific auth. `2 sources`
- [[wiki/infrastructure/dns-ssl|DNS and SSL]] — GoDaddy DDNS (cron/10min) and Let's Encrypt shared cert (19 subdomains). `1 source`
- [[wiki/infrastructure/docker-networks|Docker Networks]] — Docker Compose network topology and UFW subnet rules. `1 source`
- [[wiki/infrastructure/gcp-external-monitoring|GCP External Monitoring]] — Cloud Run monitor checks home server every 5 min, Discord alerts, DNS failover, VPN failover. `1 source`
- [[wiki/infrastructure/gluetun-vpn|Gluetun VPN]] — 3-region PIA WireGuard VPN (Toronto/Montreal/Vancouver), auto-failover, watcher service. `2 sources`
- [[wiki/infrastructure/health-api|Health API]] — Flask container exposing CPU/RAM/disk/Plex/SMART metrics to external consumers. `1 source`
- [[wiki/infrastructure/status-dashboard|Status Dashboard]] — GCP-hosted service health dashboard at status.camerontora.ca. `1 source`
- [[wiki/infrastructure/storage-raid|Storage and RAID]] — HOMENAS (8-drive software RAID5) and CAMRAID (hardware RAID5) with SMART monitoring. `1 source`

---

## Integrations

External APIs, third-party services, data flows connecting the home server to
the outside world.

_No entries yet._

---

## Decisions

Architecture decision records and significant choices.

_No entries yet._

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
