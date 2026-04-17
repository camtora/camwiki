---
title: "Decision: Dedicated OAuth2 Proxy for camerontora.ca"
type: decision
tags: [decision, adr, auth, oauth2]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Dedicated OAuth2 Proxy for camerontora.ca

**Date:** 2026-04-16 (reconstructed from repo)
**Status:** `accepted`

## Context

The shared infrastructure OAuth2 Proxy (port 4180) protects all internal services at `*.camerontora.ca` — it gates every request and only passes authenticated users through. PHASE2.md explicitly discussed consolidating all services under one unified proxy.

camerontora.ca has a fundamentally different access model: the landing page is **public by default**. Unauthenticated visitors see the map, contact form, and public service links. The Google SSO ("I'm Cam!") only unlocks the private service tiles. Routing it through the shared infra proxy would block all unauthenticated access.

## Options Considered

1. **Shared infra proxy (port 4180)** — one proxy for everything. Cleaner topology. But forces authentication for all visitors — not acceptable for a public landing page.
2. **Dedicated proxy at port 4182** — camerontora.ca manages its own OAuth2 Proxy configured for pass-through on unauthenticated requests. Private tiles gated via `x-forwarded-email` header check in application code.
3. **No proxy, app-level auth** — embed Google OAuth directly in Next.js. More code, no SSO shared with other services.

## Decision

Dedicated OAuth2 Proxy at port 4182. Configured to allow unauthenticated pass-through; app code (`page.tsx`) reads the `x-forwarded-email` header and shows/hides private tiles based on an `allowedEmails` array. The proxy adds the header when auth succeeds; when it's absent, the page renders in its public state.

## Consequences

- camerontora.ca visitors never see an auth wall — public content loads normally.
- Private service tiles are gated server-side via header check, not client-side JS.
- Two OAuth2 Proxy instances to maintain (infra proxy + this one), but they serve distinct purposes.
- The PHASE2.md "unified SSO" goal was partially achieved for the infra services — camerontora.ca is a deliberate exception due to its public-first access model.

## Related Pages

- [[wiki/infrastructure/oauth2-proxy]]
- [[wiki/projects/camerontora-ca]]

## Sources

- `raw/repos/camerontora.ca` — PHASE2.md (unified SSO discussion), CLAUDE.md (auth flow), app/page.tsx (header check)
