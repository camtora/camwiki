---
title: "Nginx Reverse Proxy"
type: infrastructure
tags: [infra, nginx, proxy, ssl]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Nginx Reverse Proxy

The central entry point for all `*.camerontora.ca` traffic. Handles SSL termination, subdomain routing, and OAuth2 authentication hand-off.

## Specification

| Property | Value |
|----------|-------|
| Container | `nginx-proxy` |
| Ports | 80 (HTTP), 443 (HTTPS) |
| Config dir | `/home/camerontora/infrastructure/nginx/conf.d/` |
| SSL cert | `/etc/letsencrypt/live/camerontora-services/` |
| Docker network | `infrastructure_default` |

## Role in the Stack

All inbound internet traffic hits Nginx first. HTTP requests are redirected to HTTPS. HTTPS requests are routed to the appropriate backend container. Protected services route through [[wiki/infrastructure/oauth2-proxy]] before reaching the backend.

```
Internet → Nginx (80/443)
              ├── Protected services → OAuth2 Proxy → backend
              └── Public services → backend directly
```

## Service Routing

### Protected (require Google SSO)

| Service | Subdomain | Backend Port |
|---------|-----------|-------------|
| Haymaker | haymaker.camerontora.ca | 3000 |
| Radarr | radarr.camerontora.ca | 7878 |
| Sonarr | sonarr.camerontora.ca | 8989 |
| Jackett | jackett.camerontora.ca | 9117 |
| Tautulli | tautulli.camerontora.ca | 8181 |
| Transmission | transmission.camerontora.ca | 9093 |
| Watchmap | watchmap.camerontora.ca | 5080 |
| Netdata | netdata.camerontora.ca | 19999 |

### Public (no authentication)

| Service | Subdomain | Backend |
|---------|-----------|---------|
| Plex | plex.camerontora.ca | 32400 |
| Seerr | seerr.camerontora.ca | 5055 |
| Who's Up | whosup.camerontora.ca | 3001 |
| Health API | health.camerontora.ca | 5000 |
| camerontora.ca | camerontora.ca | 3002 |
| Status Dashboard | status.camerontora.ca | GCP Cloud Run (CNAME) |

## Configuration Notes

- HTTP/2 enabled via `http2 on;` directive (not the deprecated `listen 443 ssl http2;` form)
- ACME challenge handler in `00-http-redirect.conf` uses `default_server` — any named server block on port 80 must include its own `/.well-known/acme-challenge/` location or certbot challenges will fail
- Custom 403 page served to authenticated users not on the allowed email list
- Reload after config changes: `docker exec nginx-proxy nginx -s reload`
- Test config: `docker exec nginx-proxy nginx -t`

## Adding a New Service

See `docs/SSO-GUIDE.md` in the infrastructure repo. Short checklist:
1. Add nginx config in `nginx/conf.d/`
2. Add subdomain to DDNS script and SSL cert (see [[wiki/infrastructure/dns-ssl]])
3. For protected services: add callback URL to Google OAuth Console
4. Reload nginx

## Connected Projects

- [[wiki/infrastructure/oauth2-proxy]]
- [[wiki/infrastructure/dns-ssl]]
- [[wiki/infrastructure/home-server]]

## Sources

- [[wiki/sources/infrastructure-repo]] — full nginx config, routing table, ACME gotcha
