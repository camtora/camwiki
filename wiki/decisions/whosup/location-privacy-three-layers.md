---
title: "Decision: Three-layer location privacy model"
type: decision
tags: [decision, adr, privacy, location, safety]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Three-layer location privacy model

**Date:** 2026-04-16 (reconstructed from docs/ARCHITECTURE.md)
**Status:** `accepted`

## Context

Who's Up shows nearby people doing activities on a map. Revealing exact GPS coordinates to all users creates stalking risk. The system needed a privacy model that enables discovery without exposing precise location to strangers.

## Options Considered

1. **No fuzzing** — show exact coordinates. Maximally useful for discovery; zero privacy protection.
2. **Show only distance, no map pin** — tell users "someone is 1.3 km away" with no visual marker. Hides location but loses the map-discovery experience entirely.
3. **Client-side fuzzing** — server sends exact coordinates, client adds noise before display. Trivially bypassed by intercepting the API response.
4. **Three-layer server-enforced model** — coarse (fuzzed) coordinates stored at creation; exact coordinates in a separate DB column; disclosure gated by approval status.

## Decision

Three-layer privacy, enforced entirely server-side:

**Layer 1 — Area-based fuzzing at write time.** When a presence is created, `latCoarse`/`lngCoarse` are computed by applying a random offset up to the area's `locationFuzzingLevel` (LOW ≈ 200m, MEDIUM ≈ 500m, HIGH ≈ 1km). `latExact`/`lngExact` store the true coordinates. Fuzzing levels are set per geographic area — Muskoka (spread-out cottages) uses HIGH; Santa Teresa (compact walkable town) uses LOW.

**Layer 2 — Coarse vs exact disclosure.** The map API (`GET /map/people`) returns only `latCoarse`/`lngCoarse` to all viewers. `latExact`/`lngExact` are only included in room details after a host approves a join request. A stranger can see approximately where you are; an approved member gets the exact meeting point.

**Layer 3 — Distance buckets.** Distances shown as `"<1km"`, `"1–5km"`, `"5–10km"`, `"10–20km"`, `"20km+"` — never an exact value. Prevents reverse-geocoding attacks (iterating distance responses to triangulate position).

Blocked users return 404 on all presence queries — doesn't leak that a presence exists.

## Consequences

- Privacy is guaranteed server-side; client-side interception cannot reveal exact location to unapproved users.
- Area fuzzing levels require manual calibration per region; a new area must have an appropriate level set.
- The `latCoarse`/`lngCoarse` fields stored at creation time — they don't change if the fuzzing algorithm is updated. Historical presences keep their original fuzz.
- Age is also calculated server-side (not sent to frontend as DOB) — same privacy-first principle applied to personal data.

## Related Pages

- [[wiki/projects/whosup]]
- [[wiki/decisions/whosup/host-approval-gate]]

## Sources

- `raw/repos/whosup/docs/ARCHITECTURE.md` — "Location Privacy Layers" section, area fuzzing diagram
- `raw/repos/whosup/docs/DATABASE.md` — Presence model (latCoarse/lngCoarse/latExact/lngExact), Area locationFuzzingLevel enum
