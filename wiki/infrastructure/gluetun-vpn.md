---
title: "Gluetun VPN"
type: infrastructure
tags: [infra, vpn, wireguard, pia, transmission]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Gluetun VPN

A VPN container (Gluetun) using PIA WireGuard that wraps Transmission. Transmission runs inside Gluetun's network namespace so all torrent traffic exits through the VPN.

## Specification

| Property | Value |
|----------|-------|
| Container | `gluetun` |
| VPN provider | Private Internet Access (PIA) |
| Protocol | WireGuard (custom provider — native PIA WG not supported in Gluetun) |
| Server | `toronto433` (`212.32.48.142`) |
| Port forwarding | Enabled; dynamic port written to `/tmp/gluetun/forwarded_port` |
| Health endpoint | `http://192.168.2.34:8090/v1/vpn/status` |
| Credentials | `docker-services/.env`: `PIA_USER`, `PIA_PASS`, `PIA_WG_PRIVATE_KEY` |

## Architecture

```
gluetun container (WireGuard tunnel → PIA toronto433)
  └── transmission (network_mode: service:gluetun)
        All traffic exits through VPN
```

## Port Forwarding

PIA assigns a dynamic forwarded port. The current port must match Transmission's `peer-port` setting in `settings.json`. If Gluetun gets a new port from PIA, `settings.json` must be updated manually.

- Check current port: `cat /home/camerontora/docker-services/gluetun/forwarded_port`
- Or: `docker logs gluetun | grep "port forward"`

## VPN Auto-Failover

The [[wiki/infrastructure/gcp-external-monitoring]] system tracks consecutive unhealthy VPN checks. After 6 consecutive failures (~30 min), it calls `health.camerontora.ca/api/health/vpn/switch` to switch to the healthiest available PIA server (highest download speed). Discord notification sent on start, completion, or failure.

Per-location nginx ports: Toronto=9091, Montreal=9092, Vancouver=9093. When VPN switches, Sonarr and Radarr download client ports are updated via API after Transmission restarts (30s wait loop ensures Transmission is ready before API calls).

## Verifying VPN

```bash
# External IP (should be PIA, not home IP)
docker exec gluetun wget -qO- https://ipinfo.io/ip

# VPN status
curl -s http://localhost:8090/v1/vpn/status

# Public IP details
curl -s http://localhost:8090/v1/publicip/ip | jq .
```

## WireGuard Key Regeneration

Keys are tied to a specific PIA server. If regeneration is needed (server issues, key expiry), see `docs/SECURITY.md` in the infrastructure repo for the full procedure: get PIA token → generate new WG keys → register with PIA API → update `.env` → restart.

## Known Issues

- PIA-assigned forwarded port renews periodically; if Gluetun gets a new port, `transmission/config/settings.json` needs `peer-port` updated
- Speedtest script has a 90s grace period after Gluetun restart (DNS takes 30–60s to initialize; too-early speedtest caused false VPN health failures)

## Connected Projects

- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/infrastructure/health-api]]

## Sources

- [[wiki/sources/infrastructure-repo]] — VPN setup, failover logic, Transmission port forwarding, WireGuard key regeneration
