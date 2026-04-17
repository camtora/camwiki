---
title: "DonorMap"
type: project
tags: [project, globalfaces, firebase, snowflake, map, firetv]
created: 2026-04-16
updated: 2026-04-16
source_count: 2
---

# DonorMap

Real-time donor visualization map for GlobalFaces Direct. Displays North American donations as animated pulsing markers on a dark-themed fixed map. Used internally and on office TV displays. Hosted on Firebase.

## Status

Current status: `active`
Last meaningful change: 2026-04-08 (data validation reference doc)

## Architecture

```
Snowflake (FIVETRAN_DATABASE) → Fivetran (sync ~15 min)
    → Cloud Function refreshDonorCache (15-min schedule)
        → Firestore cache (donorCache/us, donorCache/ca)
            → Frontend (onSnapshot real-time subscription)
```

The frontend never queries Snowflake directly. All data flows through the Firestore cache.

| Component | Tech |
|-----------|------|
| Frontend | React + TypeScript + Vite |
| Backend | Firebase Cloud Functions (TypeScript) |
| Database | Firestore (cache) + Snowflake (source of truth) |
| Map | Leaflet.js + CARTO dark tiles |
| Auth | Firebase Auth + Google SSO (domain-restricted to `@globalfacesdirect.com`) |
| Hosting | Firebase Hosting |
| TV display | Fire TV sideloaded Android app |

## Data Pipeline

### Source Tables (`FIVETRAN_DATABASE.INSIGHT_PROD_MAIN`)

| Table | Content |
|-------|---------|
| `DONORPAYMENT` | One row per payment; has CHARITYCODE, COUNTRY, POSTALCODE, CREATEDDATE |
| `DONORPAYMENTTPV` | TPV data: coordinates, product info, office |
| `DONORPAYMENTTRANSACTION` | Donor state (Approved Net, Cancelled, etc.) |
| `PRODUCT` | Product descriptions |
| `OFFICE` | Office codes |

Churn data: `HIVE_SANDBOX.TEST_ENV.MASTERDONORTABLE` — joined on `DONORPAYMENTID = "Gfd Id"` (historical views only).

### Key Filters (applied everywhere)

| Filter | Value | Reason |
|--------|-------|--------|
| `SALESCHANNELID` | `= 121` | GFD face-to-face channel only |
| `DONORSTATE` | `= 'Approved Net'` | Only approved net donors counted (applied in Cloud Function, not view) |
| `OFFICECODE` | `!= 'GFD1'` | Excludes GFD internal office |

### Date Window — 12pm EST to 12pm EST

The live view (`DONORS_LIVE_24H`) uses a 12pm-to-12pm EST window because face-to-face fundraising happens during business hours. Setting `TARGET_DATE` shows all donations from noon that day to noon the following day:

```sql
SET TARGET_DATE = '2026-04-15';
SELECT * FROM FIVETRAN_DATABASE.ANALYTICS.DONORS_LIVE_24H;
```

If `TARGET_DATE` is not set, defaults to `CURRENT_DATE()`. When today returns 0 rows, the app falls back to yesterday's data and shows a "NO DONORS ACQUIRED TODAY" banner.

### Product Name Normalization

| Raw DESCRIPTION | Normalized PRODUCTNAME |
|-----------------|----------------------|
| `Monthly`, `CHSP`, `Child Sponsorship` | `Monthly` |
| `Test OTG Range`, `OTG Fixed` | `OTG` |
| All others | Pass through as-is |

### Revenue Calculations

Revenue is **estimated** using fixed Cost of Acquisition (COA) rates — not actual payment amounts:

| Product | US | Canada |
|---------|----|--------|
| Monthly, Monthly Downgrade, Quarterly, Annual | $305 | $350 |
| OTG | Actual `PRODUCTAMOUNT` | Actual `PRODUCTAMOUNT` |

Global (combined US+CA) view converts USD to CAD at fixed rate **1.43**.

### Progress Bar Logic

The progress bar shows today's running total vs last week's total for the same day of week (`displayDate - 7 days`). The field is named `yesterdayTotal` in the code but actually holds the 7-days-ago total — this is the progress bar target.

### Comparison Columns (yd / wk / yr)

| Column | Date used |
|--------|----------|
| `yd` | `displayDate - 1 day` |
| `wk` | `displayDate - 7 days` |
| `yr` | `displayDate - 1 year` |

### Map Dots vs Count Discrepancy

Sidebar counts = all `Approved Net` donors. Map dots = only donors where coordinates exist (`LATITUDE IS NOT NULL`). Count will always be ≥ map dots.

## Snowflake Views

| View | Description |
|------|-------------|
| `DONORS_LIVE_24H` | Today's donors with 12pm EST window |
| `DONORS_HISTORY_RANGE_MULTI` | Historical donors by charity + date range |
| `CHARITIES_LAST_YEAR` | Distinct charities with activity in last year |

### Snowflake Connection

| Property | Value |
|----------|-------|
| Account | `su53882.canada-central.azure` |
| User | `donormap_service` |
| Auth | JWT (private key in GCP Secret Manager as `snowflake-private-key`) |
| Warehouse | `COMPUTE_WH` |
| Database | `FIVETRAN_DATABASE` |
| Schema | `ANALYTICS` |
| Role | `DONORMAP_READER` (SELECT only on analytics views + MASTERDONORTABLE) |

## Features

- **Real-time updates** — Firestore subscriptions with countdown timer to next refresh
- **Fixed map view** — no zoom/pan; static US, Canada, or Global view toggle
- **Pulsing markers** — animated donor dots colored by charity (brand colors extracted from logos)
- **FSA choropleth** — Canada only; colors Canadian postal regions by donor density (Statistics Canada 2021 boundary file, simplified to 1.7MB)
- **Admin settings** — charity color customization (ctora@globalfacesdirect.com only)
- **History tab** — query historical donor data by charity and date range (up to 1 year)
- **Fire TV app** — sideloaded on office TVs (TV3: 10.1.30.40, TV4: 10.1.30.42)

## FSA Region Shading (Canada)

FSA = first 3 characters of postal code (e.g., `M5V` from `M5V 2T6`). Choropleth color scale:

| Donor Count | Color |
|-------------|-------|
| > 50 | `#006d2c` (dark green) |
| 21–50 | `#31a354` |
| 11–20 | `#74c476` |
| 6–10 | `#a1d99b` |
| 1–5 | `#c7e9c0` |
| 0 | `#f7fcf5` |

Toggle is disabled when viewing US data. GeoJSON boundaries bundled with webapp (~1.7MB, ~611KB gzipped).

## Cloud Functions

| Function | Trigger | Purpose |
|----------|---------|---------|
| `refreshDonorCache` | 15-min schedule | Pulls from Snowflake → writes Firestore |
| `getDonors` | HTTP | Frontend donor query |
| `getCharityList` | HTTP | Returns charities active in last year |
| `getHistoryByCharity` | HTTP | Historical donor data for History tab |

## Fire TV App

Android app sideloaded on office Fire TVs via ADB.

### Building and Deploying (Cameron's steps)

1. **Build APK** on MacBook (via SSHFS mount):
   ```bash
   cd ~/mnt/HOMENAS/donormap/firetv-app
   export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
   COPYFILE_DISABLE=1 ./gradlew assembleDebug
   # Output: /tmp/firetv-build/outputs/apk/debug/app-debug.apk
   ```
2. **Copy APK** to server (two copies — deploy + backup):
   ```bash
   cp /tmp/firetv-build/.../app-debug.apk ~/mnt/HOMENAS/donormap/webapp/public/apk/donormap.apk
   cp /tmp/firetv-build/.../app-debug.apk ~/mnt/HOMENAS/donormap/firetv-app/releases/donormap.apk
   ```
3. **Deploy**: `cd /home/camerontora/donormap/webapp && npm run build && cd .. && firebase deploy --only hosting`
4. **Generate pairing code**: Log in at `donormap-gfd.web.app` → Pair TV → choose mode → Generate Code (6 digits, 5-min TTL)

### Install on Fire TV (Sarah's steps)

1. Enable Apps from Unknown Sources (Settings → My Fire TV → Developer Options, one-time)
2. Install Downloader app (by AFTVnews, free)
3. In Downloader, enter `donormap-gfd.web.app/apk/donormap.apk` → Install
4. Enter the 6-digit pairing code from Cameron

Fire TV stays connected permanently; pairing code only needed on first install or after factory reset.

## Adding a New Charity Logo

1. Prepare PNG with transparent background, filename = charity code (e.g., `RVHF.png`)
2. Copy to both `donormap/logos/RVHF.png` and `donormap/webapp/public/logos/RVHF.png`
3. Run `node extract-colors.js` from project root → note the hex output
4. Add brand color to `CHARITY_BRAND_COLORS` in `webapp/src/components/Footer.tsx`
5. Increment `LOGO_VERSION` constant (cache buster for browsers)
6. Deploy: `firebase deploy --only hosting`

## Infrastructure

- Fully hosted on Firebase — no home server dependency
- GFD1 office excluded from all live donor queries

## Change Log

- 2026-04-08: Data validation reference doc for engineer review
- 2026-04-04: TV install guide; ADB `pm clear` command for resetting pairing
- 2026-03-28: Global map combining US + CA data; Fire TV app icons
- Earlier: FSA choropleth, admin charity colors, History tab, TVGlobalDisplay redesign

## Key Decisions

- [[wiki/decisions/donormap/firestore-cache-intermediary|Firestore Cache Intermediary]] — 15-min scheduled Snowflake refresh; browser subscribes via real-time listeners.
- [[wiki/decisions/donormap/four-layer-domain-enforcement|Four-layer Domain Enforcement]] — OAuth hd param + frontend + Cloud Function + Firestore rules; defense-in-depth for donor data.
- [[wiki/decisions/donormap/fixed-bounds-map|Fixed-bounds Map]] — No zoom or pan; broadcast display use case; consistent view across office TVs.
- [[wiki/decisions/donormap/estimated-coa-revenue|Estimated COA Revenue]] — Fixed rates per product type; labelled EST. REVENUE; actual revenue unknown at acquisition time.
- [[wiki/decisions/donormap/twelve-noon-time-window|12pm–12pm Time Window]] — Midday-to-midday window captures full business day; eliminates timezone edge cases.
- [[wiki/decisions/donormap/firetv-pairing-code-auth|Fire TV Pairing Code Auth]] — 6-digit code + custom Firebase token; OAuth impossible on sideloaded APK.

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/donormap-repo]] — data flow, Snowflake views, Fire TV setup, office TV IPs
