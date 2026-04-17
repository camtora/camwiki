---
title: "Geospatial / PostGIS"
type: concept
tags: [concept, postgis, geospatial, postgresql, location]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Geospatial / PostGIS

PostGIS is a PostgreSQL extension that adds geospatial data types (`geometry`, `geography`), spatial indexes (GiST), and functions (`ST_DWithin`, `ST_MakePoint`, `ST_Distance`). It enables efficient radius queries — finding all rows within X metres of a point — using spatial indexing rather than full table scans.

## How It Is Used Here

[[wiki/projects/whosup\|Whosup]] has PostGIS installed but currently defers its use. All distance calculations run in application code. See [[wiki/decisions/whosup/postgis-deferred]].

| Layer | Current approach | PostGIS approach (future) |
|-------|-----------------|--------------------------|
| Coordinate storage | `Float` columns (`latCoarse`, `lngCoarse`, `latExact`, `lngExact`) | `geography` type columns |
| Radius filtering | Fetch all presences in area, filter in TypeScript | `ST_DWithin(location, ST_MakePoint(lng, lat)::geography, radius_meters)` |
| Index | `Presence(areaId, expiresAt)` B-tree | GiST spatial index |
| Distance calculation | Haversine formula in `backend/src/utils/location.ts` | `ST_Distance()` |

## Key Properties / Mechanics

### Haversine Formula

The current implementation in `location.ts`. Computes the great-circle distance between two lat/lng points on a sphere:

```
a = sin²(Δlat/2) + cos(lat1) * cos(lat2) * sin²(Δlng/2)
distance = 2R * arcsin(√a)   where R = 6371 km
```

Accurate to ~0.5% for distances under a few hundred km. Sufficient for Whosup's use case (activities within a 60km radius). The trade-off: computed in application code after fetching rows from the DB — no index assistance.

### Why PostGIS Is Installed but Not Used

At MVP scale (2–3 launch areas, small user base), a full table scan of active presences in an area is trivially fast — there are few enough rows that the Haversine loop in TypeScript takes microseconds. The migration path is well-understood but requires:
1. Schema migration: add `geography` columns alongside existing `Float` columns
2. Backfill: convert existing Float coords to geography type
3. Query rewrite: replace Haversine filter with `ST_DWithin`
4. Add GiST index
5. Remove old Float columns

This is a performance optimisation, not a correctness fix — deferred until query times become measurable.

### Location Fuzzing (Application-layer, stays in app code)

Whosup's `fuzzLocation()` utility adds random noise to coordinates before storage based on the area's `locationFuzzingLevel`:

| Level | Noise radius | Areas |
|-------|-------------|-------|
| `LOW` | ~200m | Santa Teresa |
| `MEDIUM` | ~500m | Toronto |
| `HIGH` | ~1km | Muskoka |

This is a **privacy feature** — fuzzing happens at write time so even the database doesn't store exact locations in `latCoarse`/`lngCoarse`. PostGIS cannot help here; the fuzzing logic stays in app code regardless of migration.

### Distance Buckets (Application-layer, stays in app code)

`getDistanceBucket()` converts a distance in km to a display string:

| Distance | Bucket |
|----------|--------|
| < 1 km | `"<1km"` |
| 1–5 km | `"1–5km"` |
| 5–10 km | `"5–10km"` |
| 10–20 km | `"10–20km"` |
| > 20 km | `"20km+"` |

Also a privacy feature — users never see exact distances to other users. PostGIS `ST_Distance()` would return a float; the bucket logic still runs in app code on top of that.

### `geography` vs `geometry`

PostGIS has two main spatial types:
- **`geometry`** — flat-earth (Cartesian). Fast; accurate for small areas. Uses projected coordinates (e.g., EPSG:3857).
- **`geography`** — spherical earth. Accurate for large distances. Accepts WGS84 lat/lng directly. Slightly slower but correct for queries spanning hundreds of km.

Whosup's use case (60km radius in Ontario, 15km in Costa Rica) warrants `geography` — the curvature of the earth matters at these scales.

## Relationships to Other Concepts

- [[wiki/concepts/docker-compose-networking]] — PostGIS runs inside the Whosup PostgreSQL container defined in `docker-compose.yaml`
- [[wiki/concepts/jwt-authentication]] — location data is only accessible to authenticated users with appropriate approval

## External References

- PostGIS docs: https://postgis.net/documentation/
- ST_DWithin: https://postgis.net/docs/ST_DWithin.html
