---
title: "OAuth2 Proxy"
type: infrastructure
tags: [infra, auth, sso, google, oauth]
created: 2026-04-16
updated: 2026-04-16
source_count: 2
---

# OAuth2 Proxy

Centralized Google SSO for all protected services on `camerontora.ca`. Log in once, access all protected services without re-authenticating.

## Instances

There are two OAuth2 Proxy instances running independently:

| Instance | Stack | Port | Cookie domain | Allowed users |
|----------|-------|------|---------------|---------------|
| Shared (infrastructure) | `infrastructure` | — | `.camerontora.ca` | `infrastructure/oauth2-proxy/authenticated_emails.txt` |
| camerontora.ca | `camerontoraca` | 4182 | `camerontora.ca` | `allowedEmails` array in `page.tsx` |

The shared instance in the infrastructure stack protects all `*.camerontora.ca` services (Radarr, Sonarr, Haymaker, etc.). The camerontora.ca instance is specific to the website's own auth flow ("I'm Cam!" sign-in).

## Shared Instance Specification

| Property | Value |
|----------|-------|
| Container | `oauth2-proxy` |
| Provider | Google OAuth2 |
| Cookie domain | `.camerontora.ca` |
| Allowed users | `infrastructure/oauth2-proxy/authenticated_emails.txt` |
| Docker network | `infrastructure_default` |

## Role in the Stack

Nginx forwards requests for protected services to oauth2-proxy for authentication. On success, oauth2-proxy sets a cookie scoped to `.camerontora.ca` and forwards the request to the backend. Subsequent requests from the same browser skip re-authentication as long as the cookie is valid.

## Configuration Notes

- Credentials (client ID, secret, cookie secret) stored in `infrastructure/.env`
- To add or remove allowed users, edit `authenticated_emails.txt` and reload the container
- Cookie domain `.camerontora.ca` means a single login covers all subdomains
- To force re-authentication: clear `_oauth2_proxy` cookies for `*.camerontora.ca` in browser

## Per-Service Authorization

Authentication (who can log in) is controlled by `authenticated_emails.txt`. Authorization (which services they can access) is enforced in nginx:

| Service | Access Level |
|---------|-------------|
| Haymaker | All authenticated users |
| Radarr, Sonarr, Jackett, Tautulli, Transmission, Watchmap, Netdata | Admin only |
| Health API `/api/admin/*` | Admin only |
| Plex, Seerr, Ombi | Public (no OAuth required) |
| Who's Up, camerontora.ca | Public (app-managed auth) |

Admin email mapping is in `nginx/conf.d/00-admin-map.conf` (`map $auth_email $is_admin {...}`). Adding an admin means adding a line to that map. Non-admin authenticated users get a custom 403 page (dark glassmorphism design matching the dashboard) rather than a raw error.

## Adding a New Protected Service

1. Add the nginx config routing through oauth2-proxy (see `docs/SSO-GUIDE.md`)
2. Add the callback URL to the Google OAuth Console
3. No changes needed to oauth2-proxy config itself — it's service-agnostic

## Connected Projects

- [[wiki/infrastructure/nginx-reverse-proxy]]
- [[wiki/infrastructure/home-server]]

## Sources

- [[wiki/sources/infrastructure-repo]] — shared SSO architecture and configuration
- [[wiki/sources/camerontora-ca-repo]] — camerontora.ca-specific OAuth2 instance
