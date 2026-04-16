---
title: "Transmission"
type: project
tags: [project, torrent, transmission, vpn]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Transmission

BitTorrent client running inside a Gluetun VPN container's network namespace. All torrent traffic exits through PIA WireGuard — Transmission has no direct internet access.

## Status

Current status: `active`
Last meaningful change: 2026-02-23 (port forwarding and queue fixes)

## Architecture

Transmission runs with `network_mode: service:gluetun-toronto` — it shares the network stack of the active Gluetun container rather than having its own. This means:
- All traffic exits through the VPN
- Transmission's port in `docker-compose ps` shows empty — normal, ports are exposed via Gluetun
- If Gluetun is recreated, Transmission must also be recreated (its network reference becomes stale)

## Infrastructure

- Active VPN container: `gluetun-toronto` (port 9091 externally)
- Config persisted at `docker-services/transmission/config/settings.json`
- Downloads to `/HOMENAS`
- Gluetun-watcher systemd service auto-recreates Transmission on any Gluetun restart

## Key Configuration

- Download queue: enabled, size=5 (prevents 1,749 torrents competing for 200 global peer slots)
- Port forwarding: enabled; peer port must match PIA-assigned forwarded port in `gluetun-*/piaportforward.json`
- Speedtest script has 90s grace period after Gluetun restart (DNS takes 30–60s to initialize)

## Switching VPN Region

Edit `docker-compose.yaml` — change `network_mode` and `depends_on` on the `transmission` service:

```yaml
transmission:
  network_mode: "service:gluetun-montreal"
  depends_on:
    - gluetun-montreal
```

Then: `docker-compose up -d transmission`

Available regions and ports:

| Region | Container | Web UI Port | Health Port |
|--------|-----------|-------------|-------------|
| Toronto (current) | gluetun-toronto | 9091 | 8090 |
| Montreal | gluetun-montreal | 9092 | 8091 |
| Vancouver | gluetun-vancouver | 9093 | 8092 |

After switching, [[wiki/projects/arr-suite]] (Radarr/Sonarr) download client ports must be updated to match the new port.

## Known Issues / Limitations

- PIA-assigned forwarded port renews periodically; `settings.json` peer-port must be kept in sync
- Vancouver had DNS issues (2026-01-12) — switched back to Toronto despite Vancouver being ~7x faster in speed tests
- Gluetun has a memory leak when VPN is unhealthy; all containers capped at 1GB to trigger auto-restart before OOM

## Sources

- [[wiki/sources/docker-services-repo]] — network_mode architecture, queue fix, port forwarding, VPN migration guide
- [[wiki/sources/infrastructure-repo]] — VPN auto-failover, WireGuard key management
