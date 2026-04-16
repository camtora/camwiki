---
title: "Source: whosup repo"
type: source
tags: [source, social, ios, swift, nodejs, postgis]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: whosup repo

**Raw file:** `raw/repos/whosup`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

Who's Up is a map-first social activity app: Node.js/TypeScript/Express backend with PostGIS for geospatial queries, Socket.io for real-time events, JWT + Apple/Google Sign In auth; SwiftUI iOS app targeting iOS 17+. Three launch regions with different privacy radius settings: Muskoka (high), Santa Teresa (low), Toronto (medium).

## Key Facts / Data Points

- PostgreSQL on port 5433 (non-standard — likely avoids conflict with another local Postgres instance)
- PostGIS for location-based activity discovery
- Location fuzzing until join request approved — privacy by default
- iOS app targets production URL `https://whosup.camerontora.ca/api`; nginx strips `/api` prefix
- Full docs set: API.md, DATABASE.md, DEPLOYMENT.md, ARCHITECTURE.md, IOS.md

## Pages Updated

- [[wiki/projects/whosup]] — created
