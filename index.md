# camwiki Index

Master catalog of all wiki pages. Updated by Claude on every ingest and whenever
a new page is created. Read this first when answering queries.

Last updated: 2026-04-16

---

## Projects

Home-server projects — software stacks, services, personal tools.

_No entries yet._

---

## Infrastructure

Hosts, networks, storage, operating systems, container runtimes.

- [[wiki/infrastructure/home-server|Home Server]] — Primary Ubuntu home server, port forwarding, UFW, RealVNC, secrets. `1 source`
- [[wiki/infrastructure/nginx-reverse-proxy|Nginx Reverse Proxy]] — SSL termination and subdomain routing for all *.camerontora.ca services. `1 source`
- [[wiki/infrastructure/oauth2-proxy|OAuth2 Proxy]] — Centralized Google SSO for all protected services, single cookie across *.camerontora.ca. `1 source`
- [[wiki/infrastructure/dns-ssl|DNS and SSL]] — GoDaddy DDNS (cron/10min) and Let's Encrypt shared cert (19 subdomains). `1 source`
- [[wiki/infrastructure/docker-networks|Docker Networks]] — Docker Compose network topology and UFW subnet rules. `1 source`
- [[wiki/infrastructure/gcp-external-monitoring|GCP External Monitoring]] — Cloud Run monitor checks home server every 5 min, Discord alerts, DNS failover, VPN failover. `1 source`
- [[wiki/infrastructure/gluetun-vpn|Gluetun VPN]] — PIA WireGuard VPN wrapping Transmission, port forwarding, auto-failover. `1 source`
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
