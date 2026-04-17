---
title: "Home Server"
type: infrastructure
tags: [infra, server, host]
created: 2026-04-16
updated: 2026-04-17
source_count: 2
---

# Home Server

The primary home server running all self-hosted services at `camerontora.ca`.

## Specification

| Property | Value |
|----------|-------|
| Hostname | `CAMNAS2` |
| Internal IP | `192.168.2.34` |
| OS | Ubuntu 24.04.3 LTS |
| Primary user | `camerontora` (PUID 1000 / PGID 1000) |
| Remote access | RealVNC (cloud), SSH |
| SSH | Keys only, root disabled, external port 2222 → internal 22 |

## Role in the Stack

The host for all Docker-based services. Every project (docker-services, haymaker, camerontora.ca, infrastructure) runs as a Docker Compose stack on this machine. The Nginx reverse proxy handles all inbound traffic on ports 80/443.

## Port Forwarding (Bell HomeHub Router)

| External | Internal | Service |
|----------|----------|---------|
| 2222 | 22 | SSH |
| 80 | 80 | HTTP (nginx) |
| 443 | 443 | HTTPS (nginx) |
| 32400 | 32400 | Plex (direct) |

All other ports blocked at the router.

## Firewall (UFW)

External: SSH (22), HTTP (80), HTTPS (443), Plex (32400).
Docker subnet rules (required for container→host communication):

| Subnet | Network |
|--------|---------|
| 172.17.0.0/16 | Docker bridge |
| 172.18.0.0/16 | haymaker_default |
| 172.19.0.0/16 | docker-services_default |
| 172.20.0.0/16 | camerontoraca_default |
| 172.21.0.0/16 | infrastructure_default |

New Docker networks require a corresponding UFW rule.

## Remote Access (RealVNC)

- Service: `vncserver-x11-serviced`
- Account: cameron.tora@gmail.com
- Known issue: stale cloud sessions after disconnect. Fix: run `vnc-reset`
- Auto-reset: `vnc-session-monitor` service restarts VNC 10s after disconnect
- Daily restart cron at 4am as fallback (`/etc/cron.d/vnc-maintenance`)

## Secrets Management

All secrets in `.env` files with `chmod 600`, never committed to git.

| Location | Contains |
|----------|----------|
| `infrastructure/.env` | OAuth2 credentials, API keys |
| `docker-services/.env` | Plex token, Tautulli key, PIA VPN credentials |
| `haymaker/.env` | Postgres password, Minio password |
| `camerontora.ca/.env` | Discord webhook, Tautulli API key |

## Docker Startup Mount Dependency

NAS mounts (`/HOMENAS`, `/HOMENAS2`, `/CAMRAID`) must exist before Docker starts. Enforced via a systemd drop-in:

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
echo -e "[Unit]\nRequiresMountsFor=/HOMENAS /HOMENAS2 /CAMRAID" | sudo tee /etc/systemd/system/docker.service.d/10-requires-mounts.conf
sudo systemctl daemon-reload
```

Verify: `systemctl cat docker.service` and `mount | egrep "HOMENAS|HOMENAS2|CAMRAID"`.

If Docker starts before mounts are ready, all containers that bind-mount media paths will fail to start.

## Connected Projects

- [[wiki/infrastructure/nginx-reverse-proxy]]
- [[wiki/infrastructure/docker-networks]]
- [[wiki/infrastructure/storage-raid]]
- [[wiki/infrastructure/gluetun-vpn]]

## Sources

- [[wiki/sources/infrastructure-repo]] — infrastructure repo, full topology and security config
- [[wiki/sources/desktop-camnas2-ops-reference]] — hostname, OS version, Docker systemd mount dependency (2025-12-29 snapshot)
