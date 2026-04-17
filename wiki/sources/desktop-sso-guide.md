---
title: "Source: SSO Guide"
type: source
tags: [source, infra, sso, oauth2, nginx, auth]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Source: SSO Guide

**Raw file:** `~/Desktop/SSO-GUIDE.md`
**Ingested:** 2026-04-17
**Type:** doc

## Summary

Step-by-step guide for adding new services to the camerontora.ca infrastructure with Google SSO. Covers the key architectural detail (nginx `proxy_cookie_domain` rewrite to `.camerontora.ca`) and provides three nginx config templates: protected (auth required), public (no auth), and hybrid (public with optional auth header passthrough like camerontora.ca itself).

The guide also includes a 6-step checklist, common troubleshooting scenarios (SSO not working, 401, 502, redirect_uri_mismatch, CSRF token invalid), environment variable reference, and how to generate a cookie secret.

## Key Facts / Data Points

- SSO works via shared `_oauth2_proxy` cookie scoped to `.camerontora.ca` — set by nginx `proxy_cookie_domain $host .camerontora.ca;`
- Cookie domain rewrite is in the `/oauth2/` location block — **required** for cross-subdomain SSO
- Three service patterns: protected (auth gate), public (no auth), hybrid (optional auth with `X-Forwarded-Email` passthrough)
- Allowed users: `infrastructure/oauth2-proxy/authenticated_emails.txt` — changes picked up automatically (no restart needed)
- `OAUTH2_PROXY_COOKIE_SECRET` must be 32-byte base64: `python3 -c 'import secrets; import base64; print(base64.b64encode(secrets.token_bytes(32)).decode())'`
- After adding Google OAuth callback URL, wait 1–5 minutes for Google propagation
- 6-step checklist: DNS A record → SSL cert → Google OAuth Console callback → nginx config → test → SSO cross-subdomain check

## Relevance to Wiki

- [[wiki/infrastructure/oauth2-proxy]] — nginx patterns, checklist, troubleshooting, env vars

## Contradictions or Tensions

- File structure section references `10-protected-services.conf` and `20-public-services.conf` — current nginx conf.d structure may differ; verify with `ls infrastructure/nginx/conf.d/`

## Pages Updated

- [[wiki/infrastructure/oauth2-proxy]] — nginx config patterns, adding-service procedure, troubleshooting
