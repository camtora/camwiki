---
title: "Decision: Four-layer domain enforcement for enterprise security"
type: decision
tags: [decision, adr, security, auth, firebase]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Four-layer domain enforcement for enterprise security

**Date:** 2026-04-17 (reconstructed from initial commit 1e953c0)
**Status:** `accepted`

## Context

DonorMap displays sensitive donor acquisition data for GlobalFaces Direct. Access must be restricted to `@globalfacesdirect.com` employees. A single auth check is a single point of failure — misconfiguration at any layer could expose donor data.

## Options Considered

1. **OAuth provider restriction only** — set `hd` parameter to `globalfacesdirect.com` in Google OAuth config. Simple but if OAuth config is changed, nothing else catches violations.
2. **Client-side check only** — frontend validates email domain after sign-in. Bypassable with dev tools.
3. **Four independent layers** — each layer independently enforces the domain requirement; any single misconfiguration is caught by the others.

## Decision

Four independent checks, all enforcing `@globalfacesdirect.com`:

1. **OAuth `hd` parameter** — Google OAuth provider configured to restrict to the GFD domain. First line of defence.
2. **Frontend `useAuth` hook** — validates `user.email` domain after sign-in; signs out immediately on violation.
3. **Cloud Functions** — `getDonors` and `refreshDonorCacheManual` re-validate `request.auth.token.email`; throw on mismatch.
4. **Firestore security rules** — `isGFD()` function checks `request.auth.token.email` with regex `.*@globalfacesdirect\.com$` on every read.

Defense-in-depth: a misconfigured OAuth setting, a compromised client, or a buggy Cloud Function — any single failure is caught by the remaining layers.

## Consequences

- Adding a new GFD email domain requires updating all four layers.
- Fire TV authentication cannot use this flow (sideloaded APK, no system OAuth) — requires a separate pairing code mechanism. See [[wiki/decisions/donormap/firetv-pairing-code-auth]].

## Related Pages

- [[wiki/projects/donormap]]
- [[wiki/decisions/donormap/firetv-pairing-code-auth]]

## Sources

- `raw/repos/donormap` — webapp/src/hooks/useAuth.ts, functions/src/getDonors.ts, firestore.rules, README (Security section)
