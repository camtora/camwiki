---
title: "Decision: Environment Canada MSC GeoMet API for water level data"
type: decision
tags: [decision, adr, environment-canada, water-levels, external-api]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Environment Canada MSC GeoMet API for water level data

**Date:** 2026-04-16 (reconstructed; commit 6d0112c "fix: verify EC hydrometric station IDs")
**Status:** `accepted`

## Context

The SBCA lake health dashboard shows water levels for Lake Muskoka and the South Branch Muskoka River. The app needed a reliable, automatable source for hydrometric data.

## Options Considered

1. **Web scraping** — scrape Environment Canada's Water Office website. Brittle, against ToS, high maintenance.
2. **Third-party weather API (OpenWeatherMap, etc.)** — don't provide hydrometric (water level/discharge) data; would require a separate paid service for what EC provides free.
3. **EC MSC GeoMet API** — Environment Canada's open JSON API for real-time hydrometric data. Free, no authentication, official government source.

## Decision

EC MSC GeoMet API, polled server-side daily at 8 AM ET via `node-cron`. Results stored in PostgreSQL; app reads only from DB (never directly hits EC API).

- **Station IDs** (verified active 2026-03-07):
  - Lake Muskoka at Beaumaris: `02EB018`
  - South Branch Muskoka River at Baysville: `02EB008`
- **Endpoint**: `https://api.weather.gc.ca/collections/hydrometric-realtime/items?STATION_NUMBER={id}&f=json&limit=1&sortby=-DATETIME`
- **Fields parsed**: `LEVEL`, `DISCHARGE`, `DATETIME`, `STATION_NAME`
- **Seasonal context**: backend computes `seasonLabel` (spring freshet / summer level / fall drawdown / winter low) from historical level thresholds (225.0m spring, 224.5m summer, etc.)

Server-side caching prevents the app from hammering EC API and ensures offline resilience — app shows last cached reading even if EC API is down.

Push notifications fire when 7-day delta ≥ 30 cm (±30 cm/week), de-duplicated via `sentAlerts` table.

## Consequences

- EC API is free and has no rate limits for public endpoints; zero ongoing cost.
- Station IDs must be verified periodically — EC occasionally renumbers or decommissions stations.
- Seasonal thresholds (225.0m etc.) are hardcoded from historical data; need manual update if lake management changes.
- App always shows cached data on poor Muskoka cellular connections — important for the demographic.

## Related Pages

- [[wiki/projects/sbca]]
- [[wiki/decisions/sbca/expo-push-not-firebase]]

## Sources

- `raw/repos/sbca` — backend/src/jobs/syncWaterLevels.ts, DEVELOPMENT.md (station verification URL)
