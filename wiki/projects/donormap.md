---
title: "DonorMap"
type: project
tags: [project, globalfaces, firebase, snowflake, map, firetv]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# DonorMap

Real-time donor visualization map for GlobalFaces Direct. Displays North American donations as animated pulsing markers on a dark-themed fixed map. Used internally and on office TV displays. Hosted on Firebase.

## Status

Current status: `active`
Last meaningful change: 2026-04-08 (data validation reference doc)

## Architecture

```
Snowflake (Fivetran) → Cloud Function (15-min schedule) → Firestore cache → Frontend (real-time subscription)
```

| Component | Tech |
|-----------|------|
| Frontend | React + TypeScript + Vite |
| Backend | Firebase Cloud Functions (TypeScript) |
| Database | Firestore (cache) + Snowflake (source of truth) |
| Map | Leaflet.js + CARTO dark tiles |
| Auth | Firebase Auth + Google SSO (domain-restricted to `@globalfacesdirect.com`) |
| Hosting | Firebase Hosting |
| TV display | Fire TV sideloaded app |

## Features

- **Real-time updates** — Firestore subscriptions with countdown timer to next refresh
- **Fixed map view** — no zoom/pan; static US or Canada view with toggle; Global view combining both
- **Pulsing markers** — animated donor dots colored by charity
- **Admin settings** — charity color customization (ctora@globalfacesdirect.com only)
- **History tab** — query historical donor data by charity and date range (up to 1 year back)
- **Fire TV app** — sideloaded on office TVs for ambient display (TV3: 10.1.30.40, TV4: 10.1.30.42)

## Snowflake Views

| View | Description |
|------|-------------|
| `CHARITIES_LAST_YEAR` | Distinct charities with donor activity in the last year |
| `DONORS_HISTORY_RANGE` | Historical donors filtered by charity code + date range (uses session variables) |

History tab query pattern:
```sql
SET CHARITY_CODE = 'UNICEF';
SET START_DATE = '2025-01-01';
SET END_DATE = '2025-01-31';
SELECT * FROM FIVETRAN_DATABASE.ANALYTICS.DONORS_HISTORY_RANGE WHERE DONORSTATE = 'Approved Net';
```

## Cloud Functions

| Function | Trigger | Purpose |
|----------|---------|---------|
| `refreshCache` | 15-min schedule | Pulls from Snowflake → writes Firestore |
| `getDonors` | HTTP | Frontend donor query |
| `getCharityList` | HTTP | Returns charities active in last year |
| `getHistoryByCharity` | HTTP | Historical donor data for History tab |

## Infrastructure

- Fully hosted on Firebase — no home server dependency
- Snowflake connection: `su53882.canada-central.azure` account
- Private key auth for Snowflake (`snowflake_key.p8` — not committed to git)
- GFD1 office excluded from live donor queries

## Change Log

- 2026-04-08: Data validation reference doc for engineer review
- 2026-04-04: TV install guide; ADB `pm clear` command for resetting pairing
- 2026-03-28: Global map combining US + CA data; Fire TV app icons
- Earlier: Admin charity colors, History tab, TVGlobalDisplay redesign

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/donormap-repo]] — data flow, Snowflake views, Fire TV setup, office TV IPs
