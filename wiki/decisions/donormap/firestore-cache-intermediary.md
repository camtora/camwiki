---
title: "Decision: Firestore as 15-minute refresh cache between Snowflake and the browser"
type: decision
tags: [decision, adr, firestore, snowflake, caching, real-time]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Firestore as 15-minute refresh cache between Snowflake and the browser

**Date:** 2026-04-17 (reconstructed from initial commit 1e953c0)
**Status:** `accepted`

## Context

DonorMap needs to display live donor data from Snowflake to multiple simultaneous viewers (office TV walls, staff dashboards, Fire TV). Snowflake requires JWT auth, connection pooling, and is not designed for direct browser queries.

## Options Considered

1. **Direct Snowflake queries per request** — every page load hits Snowflake. Saturates Snowflake connections; slow response times; requires exposing credentials to Cloud Functions on every request.
2. **Batch refresh → Firestore cache** — Cloud Function runs on schedule, pulls from Snowflake, writes to Firestore. Browser subscribes to Firestore via real-time listeners.

## Decision

15-minute scheduled Cloud Function (`refreshDonorCacheScheduled`) reads from Snowflake and writes to `donorCache/{us,ca}` Firestore documents. Browser subscribes via Firestore snapshot listeners.

- Firestore `donorCache` is **read-only from the frontend** — Firestore rules enforce that only Cloud Functions (via admin SDK) can write.
- Data is max 15 minutes stale, but UI updates instantly across all connected clients when the cache document updates.
- Matches the use case: face-to-face fundraising happens during business hours; 15-minute granularity is sufficient for a management dashboard.

## Consequences

- All connected clients receive the same snapshot simultaneously — consistent view across office TVs and individual dashboards.
- Manual refresh available via `refreshDonorCacheManual` Cloud Function for on-demand updates.
- Cache documents are country-scoped (`us`, `ca`) — enables Firestore rules to grant TV clients access only to their country's data.

## Related Pages

- [[wiki/projects/donormap]]
- [[wiki/concepts/snowflake]]

## Sources

- `raw/repos/donormap` — functions/src/index.ts, functions/src/refreshCache.ts, firestore.rules, README
