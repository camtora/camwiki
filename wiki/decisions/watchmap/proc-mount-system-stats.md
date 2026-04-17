---
title: "Decision: Host /proc mount for real-time system stats"
type: decision
tags: [decision, adr, monitoring, proc, docker]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Host /proc mount for real-time system stats

**Date:** 2026-04-17 (reconstructed from commit 88b1b11)
**Status:** `accepted`

## Context

Watchmap's dashboard shows real-time CPU, RAM, and load average for the home server alongside the stream map. The backend runs inside Docker; it needs access to host system metrics.

## Options Considered

1. **Docker stats API** — requires mounting the Docker socket (`/var/run/docker.sock`). Grants the container broad Docker control — a significant security concern.
2. **SSH to host** — container SSHes to the host to run `top` or `vmstat`. Complex, requires key management, adds latency.
3. **Prometheus + exporter** — production-grade metrics pipeline. Overkill for a single-server dashboard.
4. **Host /proc mount (read-only)** — mount `/proc:/host/proc:ro` into the container. Reads `/proc/stat`, `/proc/meminfo`, `/proc/loadavg` directly. Zero overhead, no security concern (read-only).

## Decision

Docker volume mount: `/proc:/host/proc:ro`. Backend reads host `/proc` files directly every 2 seconds and broadcasts `system_stats_update` via Socket.IO.

- **Accurate:** `/proc` exposes kernel-level real-time metrics — same source as `top` and `htop`.
- **Safe:** `:ro` flag — container cannot write to `/proc`.
- **Low overhead:** Text file parsing takes <1ms; 2-second polling interval is imperceptible.
- Frontend displays colour-coded thresholds: yellow at 70%, red at 90%.

## Consequences

- Linux-specific: `/proc` is a Linux kernel interface. Would not work on macOS (irrelevant since the server is Ubuntu).
- Metrics reflect the host, not the container — which is the intended behaviour for a server monitoring dashboard.

## Related Pages

- [[wiki/projects/watchmap]]

## Sources

- `raw/repos/watchmap` — backend/server.js (lines 19–20), docker-compose.yml, commit 88b1b11
