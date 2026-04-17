---
title: "OAuth2 Proxy Pattern"
type: concept
tags: [concept, auth, oauth2, sso, nginx]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# OAuth2 Proxy Pattern

OAuth2 Proxy is a reverse proxy that sits between Nginx and a backend service. It intercepts all requests, redirects unauthenticated users to an OAuth provider (Google, GitHub, etc.), and forwards authenticated requests with identity headers attached. The backend never handles auth — it just reads `X-Forwarded-Email`.

## How It Is Used Here

Two OAuth2 Proxy instances run in the home server stack. See [[wiki/infrastructure/oauth2-proxy]] for the specific configuration.

| Instance | Port | Protects | Auth scope |
|----------|------|----------|------------|
| Shared infra proxy | 4180 | Radarr, Sonarr, Jackett, Tautulli, Transmission, Netdata, Haymaker, Watchmap | `authenticated_emails.txt` allowlist |
| camerontora.ca proxy | 4182 | camerontora.ca dashboard (opt-in) | Same allowlist |

The camerontora.ca proxy is a deliberate exception — the site is **public by default** and only activates auth for specific routes that need it. See [[wiki/decisions/camerontora-ca/dedicated-oauth2-proxy]].

## Key Properties / Mechanics

### Request Flow

```
Browser → Nginx → OAuth2 Proxy (4180)
                      ├── No cookie → redirect to Google OAuth
                      ├── Valid cookie, email not in allowlist → 403 (custom page)
                      └── Valid cookie, email in allowlist → forward to backend
                                                              with X-Forwarded-Email header
```

### Why 401 = Up (for monitoring)

OAuth-protected services return `401 Unauthorized` when accessed without a valid cookie. The status dashboard treats `401` as **Up** — it proves the service is running and reachable, even though access is denied. A true outage returns a timeout or 5xx, not 401.

### Identity Headers

Authenticated requests reach the backend with:
- `X-Forwarded-Email` — the authenticated user's email address
- `X-Forwarded-User` — username portion

Haymaker reads `X-Forwarded-Email` in `get_current_user()` to auto-create users on first login. camerontora.ca's Next.js App Router reads it server-side via `headers()` — this is why App Router was chosen over Pages Router. See [[wiki/decisions/camerontora-ca/nextjs-app-router]].

### Email Allowlist

`oauth2-proxy/authenticated_emails.txt` — one email per line. Adding a user requires editing this file and reloading the proxy container. No self-serve registration. Haymaker extends this with a Discord approval flow before emails are added. See [[wiki/decisions/haymaker/multi-user-registration-flow]].

### Bypassing for Webhooks and Health Endpoints

Endpoints that must be publicly reachable (webhooks, Netdata metrics for the status dashboard, health pings) are exempted via Nginx location blocks that route directly to the backend, bypassing the proxy. See [[wiki/concepts/webhook-patterns]].

## Relationships to Other Concepts

- [[wiki/concepts/jwt-authentication]] — alternative pattern: apps like SBCA and Whosup manage their own auth without OAuth2 Proxy
- [[wiki/concepts/webhook-patterns]] — webhook endpoints must bypass the proxy

## External References

- oauth2-proxy docs: https://oauth2-proxy.github.io/oauth2-proxy/
