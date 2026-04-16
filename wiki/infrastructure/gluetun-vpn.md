---
title: "Gluetun VPN"
type: infrastructure
tags: [infra, vpn, wireguard, pia, transmission]
created: 2026-04-16
updated: 2026-04-16
source_count: 2
source_count: 1
---

# Gluetun VPN

A VPN container (Gluetun) using PIA WireGuard that wraps Transmission. Transmission runs inside Gluetun's network namespace so all torrent traffic exits through the VPN.

## Specification

| Property | Value |
|----------|-------|
| Containers | `gluetun-toronto`, `gluetun-montreal`, `gluetun-vancouver` (all run simultaneously) |
| VPN provider | Private Internet Access (PIA) |
| Protocol | WireGuard (custom provider — native PIA WG not supported in Gluetun) |
| Active server | `toronto433` (`212.32.48.142`) — current |
| Port forwarding | Enabled per-container; port stored in `gluetun-<region>/piaportforward.json` |
| Memory limit | 1GB per container (memory leak when VPN unhealthy triggers auto-restart before OOM) |
| Credentials | `docker-services/.env`: `PIA_USER`, `PIA_PASS`, `PIA_WG_TORONTO_KEY`, `PIA_WG_MONTREAL_KEY`, `PIA_WG_VANCOUVER_KEY` |

## Architecture

Three Gluetun containers run simultaneously. Transmission connects to one at a time via Docker's `network_mode`:

```
gluetun-toronto   (9091/8090) ←── Transmission (active)
gluetun-montreal  (9092/8091)
gluetun-vancouver (9093/8092)
```

Montreal and Vancouver show `Up (unhealthy)` when idle — this is expected. Only the active container needs to be healthy.

## Server Details

| Container | Region | Server | Web UI | Health |
|-----------|--------|--------|--------|--------|
| gluetun-toronto | CA Toronto | toronto433 | :9091 | :8090 |
| gluetun-montreal | CA Montreal | montreal433 | :9092 | :8091 |
| gluetun-vancouver | CA Vancouver | vancouver439 | :9093 | :8092 |

**Speed comparison (2026-01-11, 10MB test):**

| Server | Speed | vs Toronto |
|--------|-------|------------|
| Vancouver | ~282 KB/s | 7× faster |
| Montreal | ~140 KB/s | 3.5× faster |
| Toronto | ~40 KB/s | baseline |

Vancouver is fastest but had DNS issues (2026-01-12) — Toronto is current default.

## Gluetun Watcher (Auto-restart Transmission)

Transmission shares Gluetun's network namespace via `network_mode: service:gluetun-toronto`. When Gluetun is recreated (restart, update, memory limit), its container ID changes and Transmission loses network.

A systemd service auto-recreates Transmission when any Gluetun container restarts:

```bash
# Install once
sudo cp docker-services/scripts/gluetun-watcher.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now gluetun-watcher

# Check status
sudo systemctl status gluetun-watcher
```

## Port Forwarding

PIA assigns a dynamic forwarded port per container. Transmission's `peer-port` in `settings.json` must match the active container's port:

```bash
cat /home/camerontora/docker-services/gluetun-toronto/piaportforward.json
```

## VPN Auto-Failover

The [[wiki/infrastructure/gcp-external-monitoring]] system tracks consecutive unhealthy VPN checks. After 6 checks (~30 min), it calls `health.camerontora.ca/api/health/vpn/switch` to switch to the healthiest available PIA server (highest download speed). Discord notification sent on start, completion, or failure.

When VPN switches, [[wiki/projects/arr-suite]] (Sonarr/Radarr) download client ports are updated via API after Transmission restarts (30s wait loop ensures Transmission is ready).

## Verifying VPN

```bash
# Check all regions are connected
curl -s http://localhost:8090/v1/publicip/ip | jq -r '.city'  # Toronto
curl -s http://localhost:8091/v1/publicip/ip | jq -r '.city'  # Montreal
curl -s http://localhost:8092/v1/publicip/ip | jq -r '.city'  # Vancouver

# VPN status
curl -s http://localhost:8090/v1/vpn/status  # {"status":"running"}
```

## WireGuard Key Regeneration

Keys are tied to a specific PIA server. If regeneration is needed, see `docs/SECURITY.md` in the infrastructure repo: get PIA token → generate WG keys → register with PIA API → update `.env` → restart.

## Known Issues

- PIA-assigned forwarded port renews periodically; `settings.json` peer-port must be kept in sync with `piaportforward.json`
- Speedtest script has a 90s grace period after Gluetun restart (DNS takes 30–60s to initialize; too-early speedtest caused false health failures)
- Vancouver DNS issues (2026-01-12) — fastest region but unreliable; Toronto is current default despite being slowest

## Connected Projects

- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/infrastructure/health-api]]

## Sources

- [[wiki/sources/infrastructure-repo]] — VPN setup, failover logic, WireGuard key regeneration
- [[wiki/sources/docker-services-repo]] — 3-region architecture, watcher service, memory limits, speed comparison data
