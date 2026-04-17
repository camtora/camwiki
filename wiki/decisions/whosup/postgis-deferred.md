---
title: "Decision: PostGIS installed but distance calculations at application level (for now)"
type: decision
tags: [decision, adr, postgis, database, performance]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: PostGIS installed but distance calculations at application level (for now)

**Date:** 2026-04-16 (reconstructed from docs/DATABASE.md)
**Status:** `accepted` (deferred optimization)

## Context

Who's Up is a location-based app — every map query requires computing distances between the viewer and nearby presences, and filtering by radius. PostgreSQL's PostGIS extension enables efficient spatial indexes and `ST_DWithin` queries at the database layer.

## Options Considered

1. **PostGIS spatial queries** — `ST_DWithin(location, ST_MakePoint(lng, lat)::geography, radius_meters)`. Highly efficient at scale; uses spatial index (GiST) to avoid full table scan. Requires storing coordinates as `geography` type.
2. **Application-level Haversine formula** — fetch all presences in an area, compute distance in TypeScript using the Haversine formula, filter in code. Simpler SQL; no PostGIS-specific schema required.

## Decision

PostGIS extension is installed (`extensions = [postgis]` in `schema.prisma`) but not currently used. Distance calculations use a Haversine formula in `backend/src/utils/location.ts`. The `distanceKm()` and `fuzzLocation()` utilities operate on plain `Float` columns.

From `docs/DATABASE.md`: "Currently distance calculations are done in application code, but PostGIS enables efficient spatial queries if needed."

This is an explicit deferral. With 2–3 launch areas and a small initial user base, a full table scan of active presences in an area is trivially fast. The migration path (convert Float columns to `geography`, add GiST index, rewrite map queries to `ST_DWithin`) is well-understood and can be done without API changes.

## Consequences

- Map queries fetch all presences for an area then filter in code — acceptable at MVP scale; will need optimization if areas grow to thousands of concurrent presences.
- PostGIS is already installed in the Docker container — enabling it is a migration, not an infrastructure change.
- Fuzzing logic (`fuzzLocation()`) stays in application code regardless — PostGIS doesn't help there.
- Distance bucket logic (`getDistanceBucket()`) also stays in app code — intentional (buckets are a privacy feature, not a query feature).

## Related Pages

- [[wiki/projects/whosup]]
- [[wiki/decisions/whosup/location-privacy-three-layers]]

## Sources

- `raw/repos/whosup/docs/DATABASE.md` — PostGIS note, schema setup commands
- `raw/repos/whosup` — backend/prisma/schema.prisma, backend/src/utils/location.ts
