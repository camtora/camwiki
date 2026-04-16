---
title: "Docker Networks"
type: infrastructure
tags: [infra, docker, networking]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Docker Networks

All services are isolated into Docker Compose networks. Inter-service communication uses `host.docker.internal`; UFW rules permit each subnet.

## Networks

| Network | Subnet | Services |
|---------|--------|----------|
| `docker-services_default` | 172.19.0.0/16 | plex, radarr, sonarr, jackett, tautulli, seerr, bazarr, tdarr, flaresolverr, watchmap, gluetun; transmission inside gluetun namespace |
| `infrastructure_default` | 172.21.0.0/16 | nginx-proxy, oauth2-proxy, health-api |
| `haymaker_default` | 172.18.0.0/16 | db, minio, api, web |
| `camerontoraca_default` | 172.20.0.0/16 | camerontora_web |
| Docker bridge | 172.17.0.0/16 | default bridge |

## Key Points

- Services bind ports to `0.0.0.0`; UFW handles external blocking (not Docker itself)
- Each Docker network requires a corresponding UFW allow rule for the subnet — adding a new Compose stack means adding a new UFW rule
- nginx-proxy reaches backends via `host.docker.internal` or direct container names within `infrastructure_default`

## Connected Projects

- [[wiki/infrastructure/home-server]]
- [[wiki/infrastructure/nginx-reverse-proxy]]

## Sources

- [[wiki/sources/infrastructure-repo]] — network topology, UFW subnet rules
