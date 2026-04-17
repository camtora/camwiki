---
title: "Decision: Fire TV pairing code authentication with custom Firebase tokens"
type: decision
tags: [decision, adr, firetv, android, auth, firebase]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Fire TV pairing code authentication with custom Firebase tokens

**Date:** 2026-04-17 (reconstructed from commit 012987e)
**Status:** `accepted`

## Context

DonorMap runs on office TV walls as a sideloaded Fire TV APK (WebView app). Fire TV devices cannot complete Google OAuth — no system browser, no Chrome, no Play Services OAuth flow. TVs need to stay authenticated indefinitely once paired, without daily password prompts.

## Options Considered

1. **Google OAuth** — the standard GFD domain auth flow. Impossible: sideloaded APK has no system browser for the OAuth redirect.
2. **Hardcoded API key in APK** — all TVs share one key. Insecure; unrevocable without a new APK deploy; no per-device audit trail.
3. **Device ID + PIN** — complex to revoke; no time-based expiry.
4. **6-digit pairing code + custom Firebase token** — staff generate a short-lived code from the web app; TV user enters it once via D-pad; Cloud Function issues a custom Firebase auth token with country and `isTV` claims.

## Decision

Pairing code flow:

1. Staff generate a **6-digit code** (5-minute validity, one-time use) from `PairTVModal` in the web app
2. TV user enters code via D-pad-navigable keypad (`TVPairingScreen`)
3. `redeemTVCode` Cloud Function verifies code, issues a **custom Firebase auth token** with `{ isTV: true, country: "us"|"ca" }` claims
4. Firestore security rules use `isTV()` helper for country-scoped `donorCache` read access
5. Token is cached on the TV — persists across reboots; no daily re-auth needed

Code is temporary (5 min) and single-use. Each TV gets its own token with explicit country scoping. Tokens can be revoked via Firebase console.

## Consequences

- Pairing is a one-time manual step per TV — low friction for initial setup.
- `isTV` tokens bypass the `@globalfacesdirect.com` domain check — their access is deliberately narrower (read-only `donorCache`, country-scoped). See [[wiki/decisions/donormap/four-layer-domain-enforcement]].
- Custom tokens cannot be refreshed automatically — if a token expires (Firebase default: 1 hour for ID tokens), TV must be re-paired. Mitigated by caching the custom token and re-signing in silently.

## Related Pages

- [[wiki/projects/donormap]]
- [[wiki/decisions/donormap/four-layer-domain-enforcement]]
- [[wiki/decisions/donormap/fixed-bounds-map]]

## Sources

- `raw/repos/donormap` — functions/src/tvPairing.ts, functions/src/index.ts, webapp/src/components/PairTVModal.tsx, firetv-app/MainActivity.kt, firestore.rules, commit 012987e
