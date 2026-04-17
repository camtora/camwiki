---
title: "Decision: Transmission inside Gluetun network namespace"
type: decision
tags: [decision, adr, vpn, transmission, docker]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Transmission inside Gluetun network namespace

**Date:** 2026-01-11 (from changelog)
**Status:** `accepted`

## Context

Transmission should only send traffic through the VPN — not via the home server's public IP. Docker offers several approaches to enforce this. The requirement was: if the VPN drops, Transmission loses connectivity entirely (kill-switch behavior), rather than falling back to the cleartext connection.

## Options Considered

1. **Transmission's built-in VPN bind** — configure Transmission to bind its network interface to the VPN adapter IP. Works, but fragile: if the VPN interface name changes, Transmission silently falls back to the default route.
2. **Host-level VPN** — run PIA client on the host; all traffic VPN-routed. Affects all containers and host processes, not just Transmission. Can't scope it to one service.
3. **`network_mode: service:gluetun-toronto`** — Transmission shares Gluetun's network namespace entirely. Transmission has no network interface except Gluetun's. If Gluetun is stopped, Transmission has no network at all.

## Decision

`network_mode: "service:gluetun-toronto"` in `docker-compose.yaml`. Transmission is placed inside Gluetun's network namespace. This is the strictest possible isolation: Transmission cannot reach the internet except through the VPN tunnel. Kill-switch behavior is automatic — Gluetun down = Transmission offline.

The trade-off is coupling: when Gluetun is recreated (update, memory limit trigger), Transmission's network reference becomes stale and it must also be recreated. This is handled by the `gluetun-watcher` systemd service, which auto-recreates Transmission whenever any Gluetun container restarts.

## Consequences

- Transmission shows no ports in `docker-compose ps` — normal, ports are exposed via Gluetun.
- After a Gluetun restart, Transmission must be recreated: `docker-compose up -d transmission`. The watcher service handles this automatically.
- Switching VPN regions requires updating the `network_mode` value in `docker-compose.yaml` and recreating Transmission.
- Three Gluetun containers (toronto/montreal/vancouver) are kept running; `network_mode` points to whichever region is active.

## Related Pages

- [[wiki/projects/transmission]]
- [[wiki/infrastructure/gluetun-vpn]]
- [[wiki/projects/docker-services]]

## Sources

- `raw/repos/docker-services` — docker-compose.yaml, docs/VPN-MIGRATION.md (network_mode explanation), docs/UPDATING-CONTAINERS.md
