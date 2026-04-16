---
title: "OAuth2 Proxy"
type: infrastructure
tags: [infra, auth, sso, google, oauth]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# OAuth2 Proxy

Centralized Google SSO for all protected services on `camerontora.ca`. Log in once, access all protected services without re-authenticating.

## Specification

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

## Adding a New Protected Service

1. Add the nginx config routing through oauth2-proxy (see `docs/SSO-GUIDE.md`)
2. Add the callback URL to the Google OAuth Console
3. No changes needed to oauth2-proxy config itself — it's service-agnostic

## Connected Projects

- [[wiki/infrastructure/nginx-reverse-proxy]]
- [[wiki/infrastructure/home-server]]

## Sources

- [[wiki/sources/infrastructure-repo]] — SSO architecture and configuration
