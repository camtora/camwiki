---
title: "Decision: Next.js App Router for camerontora.ca"
type: decision
tags: [decision, adr, nextjs, frontend]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Next.js App Router for camerontora.ca

**Date:** 2026-04-16 (reconstructed from repo)
**Status:** `accepted`

## Context

camerontora.ca is a personal landing page, not a complex web app. A static site generator (Astro, Hugo) or a simple SPA would cover the public-facing content. The question was whether Next.js with App Router was justified.

## Options Considered

1. **Static site generator (Astro, Hugo)** — fast, simple, no server. But: can't read request headers server-side (needed for auth), and can't host API routes for Tautulli/Discord without a separate backend.
2. **SPA (Vite + React)** — works for the map and tiles, but auth header reading requires SSR. Would need a separate API server for Tautulli, Discord webhooks, and the status banner.
3. **Next.js App Router** — SSR enables server components to read `x-forwarded-email` header directly. Built-in API routes (`/api/tautulli`, `/api/contact`, `/api/plex-access-request`, `/api/status`) eliminate a separate backend entirely.

## Decision

Next.js 15 with App Router. The SSR requirement is the deciding factor: the auth model depends on reading `x-forwarded-email` from OAuth2 Proxy on the server side — a pure SPA cannot do this without a backend API to proxy through. App Router server components make it a single-step render with no client-side auth logic.

## Consequences

- No separate backend needed for Tautulli integration, Discord webhooks, or status banner.
- Auth is handled server-side — private tiles are never sent to unauthorized clients even in the initial HTML.
- Next.js is heavier than a static site; acceptable given it runs on a home server with no traffic constraints.
- Consistent with other projects in the stack (Haymaker frontend is also Next.js).

## Related Pages

- [[wiki/projects/camerontora-ca]]

## Sources

- `raw/repos/camerontora.ca` — CLAUDE.md (architecture overview), app/app/page.tsx (server component auth), app/app/api/ (API routes)
