---
title: "Decision: 12pm–12pm EST time window instead of midnight-to-midnight"
type: decision
tags: [decision, adr, time-window, business-logic, timezone]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: 12pm–12pm EST time window instead of midnight-to-midnight

**Date:** 2026-04-17 (reconstructed from initial commit 1e953c0)
**Status:** `accepted`

## Context

DonorMap shows a "today's donors" count and revenue figure. The time window definition determines what "today" means. Door-to-door fundraising happens during business hours; no donors are acquired at 2am.

## Options Considered

1. **Midnight UTC** — database default. Creates timezone confusion: East Coast sees results until 8pm EST the previous day; West Coast donors from 5pm prior day appear in "today."
2. **Midnight EST** — correct for Eastern timezone but still spans two nights of zero activity.
3. **12pm EST to 12pm EST** — midday to midday window captures a full business day, regardless of timezone. Same comparison duration for yesterday and today.

## Decision

12pm EST → 12pm EST next day window, implemented via `DATEADD(hour, 12, ...)` and `DATEADD(hour, 36, ...)` in the Snowflake view (`DONORS_LIVE_24H`) and replicated in `functions/src/snowflake.ts`.

Documented in `docs/data-validation.md`: *"Why 12pm to 12pm? Face-to-face fundraising happens during business hours. The 12pm window captures a full working day regardless of timezone edge cases."*

Same window is used for "yesterday" comparison — making day-over-day comparisons apples-to-apples (both 12pm windows, same duration).

## Consequences

- "Today" at 11am EST refers to yesterday's afternoon through this morning — slightly unintuitive but correct for the use case.
- Any new query against donor data must use the same `DATEADD` logic or comparisons will drift.
- Time window logic is baked into the Snowflake SQL view — changing it requires a view update and cache flush.

## Related Pages

- [[wiki/projects/donormap]]
- [[wiki/decisions/donormap/firestore-cache-intermediary]]

## Sources

- `raw/repos/donormap` — donor_view.sql (lines 42–45), functions/src/snowflake.ts, docs/data-validation.md (lines 64–74)
