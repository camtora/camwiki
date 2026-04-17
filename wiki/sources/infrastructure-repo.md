---
title: "Source: infrastructure repo"
type: source
tags: [source, infrastructure, networking, security, monitoring]
created: 2026-04-16
updated: 2026-04-17
source_count: 1
---

# Source: infrastructure repo

**Raw file:** `raw/repos/infrastructure`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

The infrastructure repo is the central configuration and documentation hub for all `*.camerontora.ca` services running on the home server. It manages the Nginx reverse proxy, OAuth2 SSO, SSL certificates, DNS, and several monitoring systems.

The repo is organized around a Docker Compose stack (nginx-proxy + oauth2-proxy) with supporting scripts, configs, and a health API container. Alongside the running services, a GCP-hosted external monitor polls the home server every 5 minutes and alerts via Discord — solving the fundamental problem that local monitors can't alert when the home internet is down.

Two reliability systems stand out: DNS failover (auto-switches public DNS to GCP static IP when internet drops) and VPN auto-failover (switches Transmission's PIA WireGuard server if the current one is unhealthy for 30+ minutes). Both are implemented in the GCP Cloud Run monitor and coordinated with the home server via the health API.

Storage monitoring covers HOMENAS (8-drive software RAID5) and CAMRAID (hardware RAID5), with per-drive SMART data exposed through the health API and surfaced in the status dashboard.

## Key Facts / Data Points

- Home server IP: `192.168.2.34`; public ports forwarded: 80, 443, 32400, SSH on 2222→22
- DDNS: GoDaddy, cron every 10 min, ~8,640 API calls/month (well under 20k limit)
- SSL: Let's Encrypt `camerontora-services` cert, webroot mode, shared across 19 subdomains
- GCP project: `cameron-tora`; Cloud Run monitor: `home-monitor` in `us-central1`
- External monitor schedule: `*/5 * * * *` via Cloud Scheduler
- VPN: PIA WireGuard, server `toronto433` (`212.32.48.142`), port forwarding enabled
- Transmission runs inside Gluetun network namespace; PIA-assigned port written to `/tmp/gluetun/forwarded_port`
- Secrets: all in `.env` files chmod 600; never committed to git
- RealVNC for remote desktop (account: cameron.tora@gmail.com); auto-reset service handles stale cloud sessions
- UFW allows: 22, 80, 443, 32400, plus Docker subnets 172.17-21.0.0/16
- Ombi decommissioned (2026-03-28), replaced by Seerr; static migration page remains on home server

## Relevance to Wiki

First infrastructure ingest. Establishes the core networking, security, and monitoring topology that all other projects build on.

## Contradictions or Tensions

None — first ingest.

## Pages Updated

- [[wiki/infrastructure/home-server]] — created
- [[wiki/infrastructure/nginx-reverse-proxy]] — created
- [[wiki/infrastructure/oauth2-proxy]] — created; jshor96 access added 2026-04-17
- [[wiki/infrastructure/gcp-external-monitoring]] — created
- [[wiki/infrastructure/health-api]] — created
- [[wiki/infrastructure/status-dashboard]] — created; Wiki Q&A panel + Plex banner added 2026-04-17
- [[wiki/infrastructure/gluetun-vpn]] — created
- [[wiki/infrastructure/dns-ssl]] — created
- [[wiki/infrastructure/storage-raid]] — created
- [[wiki/infrastructure/docker-networks]] — created
- [[wiki/concepts/dns-failover]] — created
