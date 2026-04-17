---
title: "OAuth2 Proxy"
type: infrastructure
tags: [infra, auth, sso, google, oauth]
created: 2026-04-16
updated: 2026-04-17
source_count: 3
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

## The Cookie Domain Trick

OAuth2 Proxy sets cookies per-subdomain by default, which breaks SSO across subdomains. The fix is in every nginx service config:

```nginx
proxy_cookie_domain $host .camerontora.ca;
```

This rewrites `haymaker.camerontora.ca` → `.camerontora.ca`, making the `_oauth2_proxy` cookie valid across all subdomains. **This line must appear in the `/oauth2/` location block — missing it is the most common SSO bug.**

## Adding a New Protected Service

**6-step checklist:**
1. Add DNS A record pointing to the server IP
2. Add subdomain to SSL cert if not using wildcard: `sudo certbot certonly --expand -d newservice.camerontora.ca`
3. Add callback URL to Google OAuth Console: `https://newservice.camerontora.ca/oauth2/callback` — wait 1–5 min for propagation
4. Create nginx config at `infrastructure/nginx/conf.d/XX-newservice.conf` (see pattern below)
5. Test and reload: `docker exec nginx-proxy nginx -t && docker exec nginx-proxy nginx -s reload`
6. Test SSO: log in to any existing service, then navigate to new service — should skip auth prompt

### Nginx Config Patterns

**Protected (auth required):**
```nginx
server {
    listen 443 ssl http2;
    server_name newservice.camerontora.ca;
    ssl_certificate /etc/letsencrypt/live/camerontora-services/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/camerontora-services/privkey.pem;

    location /oauth2/ {
        proxy_pass http://oauth2-proxy;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cookie_domain $host .camerontora.ca;   # ← SSO key
    }
    location = /oauth2/auth {
        internal;
        proxy_pass http://oauth2-proxy/oauth2/auth;
        proxy_set_header Content-Length "";
        proxy_pass_request_body off;
    }
    location @error401 {
        return 302 https://$host/oauth2/start?rd=$scheme://$host$request_uri;
    }
    location / {
        auth_request /oauth2/auth;
        auth_request_set $auth_email $upstream_http_x_auth_request_email;
        error_page 401 = @error401;
        proxy_pass http://host.docker.internal:YOUR_PORT;
        proxy_set_header X-Forwarded-Email $auth_email;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

**Public (no auth):** Omit all oauth2 blocks; proxy directly to `host.docker.internal:PORT`.

**Hybrid (public with optional auth passthrough):** Use `error_page 401 = @public;` instead of `@error401`, and add a `location @public` block that proxies without `auth_request`. The `X-Forwarded-Email` header is set if logged in, empty if not. This is how camerontora.ca itself works.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Prompted to log in on each subdomain | Check `proxy_cookie_domain $host .camerontora.ca;` is in `/oauth2/` block; clear `_oauth2_proxy` cookies |
| 401 Unauthorized | Check email is in `authenticated_emails.txt`; check oauth2-proxy logs |
| 502 Bad Gateway | Check oauth2-proxy container is running; check backend port is correct |
| `redirect_uri_mismatch` | Add `https://subdomain.camerontora.ca/oauth2/callback` to Google OAuth Console; wait 1–5 min |
| CSRF Token Invalid | Ensure `proxy_cookie_domain` is set in `/oauth2/` block; clear cookies |

## Connected Projects

- [[wiki/infrastructure/nginx-reverse-proxy]]
- [[wiki/infrastructure/home-server]]

## Sources

- [[wiki/sources/infrastructure-repo]] — shared SSO architecture and configuration
- [[wiki/sources/camerontora-ca-repo]] — camerontora.ca-specific OAuth2 instance
- [[wiki/sources/desktop-sso-guide]] — nginx config patterns, 6-step checklist, troubleshooting guide
