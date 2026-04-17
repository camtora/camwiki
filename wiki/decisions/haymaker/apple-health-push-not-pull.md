---
title: "Decision: Apple Health via push webhook (Health Auto Export) not direct pull"
type: decision
tags: [decision, adr, apple-health, integration, webhooks]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Apple Health via push webhook (Health Auto Export) not direct pull

**Date:** 2026-04-16 (reconstructed from docs/apple-health-integration.md)
**Status:** `accepted`

## Context

Haymaker needs Apple Watch health data: sleep stages, steps, workout HR/calories, and nutrition from MyFitnessPal. Withings covers weight and body composition via a pull model (OAuth2 flow, server-side API calls). Apple Health needed a different approach.

Apple does not expose a public server-side API for Health data. All Apple Health access is device-local or requires an iOS app on the user's device.

## Options Considered

1. **Build a native iOS app** — write a Swift app that reads HealthKit and syncs to Haymaker. Full control, no third-party dependency. But: significant iOS development effort for a personal tool; app requires Apple Developer Program ($99/year) to deploy to personal device reliably.
2. **Apple Health XML export** — export Health data as XML and import manually. No automation; manual friction every day.
3. **Health Auto Export (third-party iOS app, ~$5)** — existing App Store app that reads HealthKit and POSTs to any REST endpoint on a schedule. Push model: Haymaker receives data via webhook, no polling needed.

## Decision

Health Auto Export (push model). The user runs the app on their iPhone; it posts JSON to `POST /api/webhooks/apple-health` on an hourly schedule. The endpoint authenticates with a per-user API key stored in `UserSettings.apple_health_api_key`.

The endpoint is intentionally **public** (no OAuth2 Proxy) — it uses API key authentication (`X-API-Key` header) because the webhook caller is the Health Auto Export app on iPhone, which cannot go through the browser-based OAuth2 flow.

Push vs pull matters here: Apple's server-side API does not exist. The only way to get the data server-side is to have the device push it.

## Consequences

- One-time $5 purchase; no recurring cost.
- Setup requires manual configuration in Health Auto Export (URL, API key, metrics selection, schedule).
- Generates one API key per Haymaker user; stored in `UserSettings` and rotatable from Settings UI.
- Data lands in separate `apple_health_*` tables (nutrition, sleep, steps, workouts, meal_entries) — kept isolated from manually entered data; used primarily for analytics and auto-fill suggestions.
- Apple Watch sleep stages (REM, deep, core) are required for the full sleep score formula; without them only the duration component applies. This is a Withings API limitation too — Apple Watch sleep data is not available via Withings API.
- Regularity score and effort score are computed server-side from the pushed data (see [[wiki/projects/haymaker]] for formulas).

## Related Pages

- [[wiki/projects/haymaker]]
- [[wiki/decisions/haymaker/manual-data-priority]]

## Sources

- `raw/repos/haymaker/docs/apple-health-integration.md` — full architecture, data schemas, score formulas, setup guide
- `raw/repos/haymaker` — apps/api/app/routes.py (webhook endpoint), apps/api/app/models.py (apple_health_* tables)
