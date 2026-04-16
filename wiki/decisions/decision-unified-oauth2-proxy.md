---
title: "Decision: Unified OAuth2 Proxy for SSO"
type: decision
tags: [decision, adr, auth, sso, oauth, infrastructure]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Unified OAuth2 Proxy for SSO

**Date:** 2026-01-13 (approx, "Phase 3: SSO migration" in commit history)
**Status:** `accepted`

## Context

Protected services (Radarr, Sonarr, Jackett, Tautulli, Transmission, Watchmap, Netdata, Haymaker) each needed authentication to prevent public access. The options were per-service authentication (each service manages its own login) or a centralized proxy.

Previously, some services used HTTP basic auth (`.htpasswd` via nginx) and Haymaker had its own OAuth2 Proxy instance. This meant separate credentials per service, no single sign-on, and maintenance overhead when adding new services.

## Options Considered

1. **Per-service auth** — each service manages its own login (basic auth, service-specific accounts). Simple but no SSO; adding a new service requires new credentials.
2. **Shared OAuth2 Proxy** — single proxy in front of all protected services, cookie scoped to `.camerontora.ca`, backed by Google SSO. Log in once, access all services.

## Decision

Deploy a single OAuth2 Proxy instance in the infrastructure stack. Nginx routes all protected services through it. The session cookie is scoped to `.camerontora.ca` — one Google login covers all subdomains.

Allowed users are maintained in `oauth2-proxy/authenticated_emails.txt`. Adding a new protected service only requires an nginx config change and a Google OAuth Console callback URL — no new credentials to manage.

## Consequences

- Single login for all protected services — no re-authentication between Radarr, Sonarr, Haymaker, etc.
- Custom 403 page for authenticated users who aren't on the allowlist
- Removing access is a single-line edit to `authenticated_emails.txt`
- New protected services are trivial to add — nginx config + OAuth Console callback
- camerontora.ca maintains a second, separate OAuth2 Proxy instance (port 4182) for its own website sign-in flow — this is a distinct concern from infrastructure-level service protection

## Related Pages

- [[wiki/infrastructure/oauth2-proxy]]
- [[wiki/infrastructure/nginx-reverse-proxy]]

## Sources

- [[wiki/sources/infrastructure-repo]] — SSO-GUIDE.md, SECURITY.md, commit history
