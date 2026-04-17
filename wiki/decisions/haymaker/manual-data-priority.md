---
title: "Decision: Manual entry takes precedence over auto-synced data"
type: decision
tags: [decision, adr, data-model, integrations, ux]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Manual entry takes precedence over auto-synced data

**Date:** 2026-04-16 (reconstructed; from commit fef4490 and code comments)
**Status:** `accepted`

## Context

Haymaker has two data sources for most health metrics: manually entered data (via daily check-in, meals, habit logs) and auto-synced data from Apple Health and Withings. The question was which source should win when both exist, and how to present them in the dashboard and analytics.

## Options Considered

1. **Auto-synced data takes priority** — integrations provide more reliable, more frequent data; prefer them. Risk: a sensor error, a misclassified nap, or a bad Withings measurement silently corrupts the primary record.
2. **Latest entry wins** — merge strategy; whichever was written most recently is authoritative. Risk: background syncs overwrite deliberate manual corrections.
3. **Manual entry takes priority; auto-sync supplements** — manually entered data is always authoritative for the primary dashboard. Auto-synced data is used for analytics pages and as fill-in when no manual entry exists.

## Decision

Manual data is always authoritative for the primary dashboard and daily check-in (`/check-in`, `/mobile`). From commit `fef4490`: "Dashboard: use manual data only, extend to 30 days."

Auto-synced data (Apple Health, Withings) is:
- **Supplemental for analytics** — `/analytics/workouts`, `/analytics/recovery`, `/analytics/metabolic` use Apple Health data because it has more granular workout/sleep detail than manual entry
- **Available for auto-fill** — "Pull from Apple Health" / "Pull from Withings" buttons on check-in pages let the user explicitly accept a sync into the manual record
- **Displayed separately** — integration data is shown in context (e.g., Apple Health sleep score alongside manual sleep hours entry) rather than overwriting it

## Why

Personal health data requires user agency. An auto-sync error (Withings returning a stale reading, nap mis-tagged as full sleep, Apple Watch HR sensor spike) should never silently corrupt the user's own record. The act of entering data manually is also intentional — it creates reflection and accountability that auto-sync removes.

Code comment in routes: "Manual data only; Apple Health supplements analytics."

The integration buttons ("Pull from Apple Health") are designed to be explicit: three states — data available (active), data synced (disabled, confirmation), no data (disabled, no entry) — so the user always understands what's happening.

## Consequences

- Check-in pages show both manual entry fields and integration-sourced suggestions side-by-side; user decides what to commit.
- Dashboard and 30-day trends use only manually committed data — avoids noisy auto-sync skewing trend lines.
- Analytics pages can use raw Apple Health data freely since they are exploratory, not record-of-record.
- Users who want full automation can always click "Pull from Apple Health" without reviewing; the friction is minimal for power users and protective for everyone.

## Related Pages

- [[wiki/projects/haymaker]]
- [[wiki/decisions/haymaker/apple-health-push-not-pull]]

## Sources

- `raw/repos/haymaker` — commit fef4490 ("Dashboard: use manual data only"), apps/api/app/routes.py (integration endpoints), apps/web/src/app/check-in/page.tsx (auto-fill button states)
