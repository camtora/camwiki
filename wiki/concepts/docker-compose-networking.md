---
title: "Docker Compose Networking"
type: concept
tags: [concept, docker, networking, vpn, containers]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Docker Compose Networking

Docker Compose creates virtual networks that containers join by name. Containers on the same network reach each other by service name (DNS). Containers on different networks are isolated. Network namespace sharing via `network_mode: service:` goes further — two containers share the same network stack entirely.

## How It Is Used Here

The home server uses two Docker Compose files with distinct network strategies:

| File | Network | Key pattern |
|------|---------|-------------|
| `docker-services/docker-compose.yaml` | `infrastructure_default` (bridge) | Service-name routing; Gluetun VPN namespace sharing |
| `whosup/docker-compose.yaml` | Internal bridge | Backend + DB isolation |
| `sbca/infrastructure/docker-compose.yaml` | Internal bridge | API + Postgres + MinIO |

All services that need external access go through [[wiki/infrastructure/nginx-reverse-proxy]] on the `infrastructure_default` network.

## Key Properties / Mechanics

### Default Bridge Network

Every `docker-compose.yaml` creates a default bridge network named `<project>_default`. Containers on the same network reach each other by service name:

```yaml
# backend can reach db at http://db:5432
services:
  backend:
    networks: [default]
  db:
    networks: [default]
```

External services (like nginx-proxy) join by declaring the same network as external:

```yaml
networks:
  infrastructure_default:
    external: true
```

### `network_mode: service:` — Namespace Sharing

The most powerful (and unusual) pattern in this stack. Instead of a container having its own network namespace, it inherits another container's entirely — same IP, same ports, same routing table.

```yaml
services:
  gluetun-toronto:
    # VPN container — all traffic exits through WireGuard tunnel
    
  transmission:
    network_mode: "service:gluetun-toronto"
    depends_on: [gluetun-toronto]
```

**Effect:** Transmission has no network interface of its own. All its traffic — inbound and outbound — flows through Gluetun's WireGuard tunnel. If the VPN drops, Transmission loses connectivity entirely. This is a **hardware kill-switch** at the kernel level, not a firewall rule. See [[wiki/decisions/docker-services/transmission-gluetun-network-mode]].

**Consequence:** Nginx cannot reach Transmission by service name. It must reach it via `gluetun-toronto:<port>` since Transmission's port is exposed on Gluetun's namespace, not its own.

### `depends_on` and Startup Order

`depends_on` ensures a service starts after its dependency, but does not wait for the dependency to be *healthy*. For the Gluetun/Transmission pattern, a `gluetun-watcher` systemd service monitors Gluetun and recreates Transmission if the VPN restarts — because `depends_on` alone doesn't handle mid-run failures.

### Port Conflicts

When multiple services expose the same internal port, host port mapping distinguishes them:

```yaml
postgres-whosup:
  ports: ["5433:5432"]   # host 5433 → container 5432

postgres-sbca:
  ports: ["5434:5432"]   # host 5434 → container 5432
```

Whosup's PostgreSQL runs on host port 5433 (non-standard) to avoid conflicts with other Postgres instances.

### UFW and Docker

Docker bypasses UFW by writing directly to `iptables`. Subnet-level UFW rules (e.g., allowing the `172.x.x.x/16` Docker subnet) are needed to control container-to-host or inter-network traffic. See [[wiki/infrastructure/docker-networks]].

## Relationships to Other Concepts

- [[wiki/concepts/oauth2-proxy-pattern]] — the proxy pattern depends on containers sharing the `infrastructure_default` network
- [[wiki/concepts/gcp-cloud-run]] — contrast: Cloud Run uses managed networking with no compose files

## External References

- Docker Compose networking: https://docs.docker.com/compose/networking/
- network_mode: https://docs.docker.com/compose/compose-file/compose-file-v3/#network_mode
