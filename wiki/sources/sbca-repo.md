---
title: "Source: sbca repo"
type: source
tags: [source, mobile, expo, fastify, stripe, muskoka, accessibility]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: sbca repo

**Raw file:** `raw/repos/sbca`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

SBCA is a React Native (Expo SDK 55) + Fastify monorepo for the Stephens Bay Cottage Association. Three features: membership portal (Stripe dues), lake health dashboard (Environment Canada water levels + community observations), events and community board. Admin portal (Next.js 14, Phase 3) is in progress. Accessibility is a hard requirement — primary demographic is 55–75.

## Key Facts / Data Points

- Environment Canada water level stations: Beaumaris `02EB018`, Baysville `02EB008`
- Stripe CardField only (no Apple Pay/Google Pay) — PaymentIntent created and verified server-side
- MinIO on ports 9010 (API) / 9011 (console) for observation photos
- PostgreSQL on port 5432 (standard); API on port 3003
- Expo Router file-based routing; requires `xcworkspace` not `xcodeproj` after prebuild
- Bundle identifier: `com.camerontora.sbca`
- Admin portal (Next.js 14) commits suggest it is at least partially live

## Pages Updated

- [[wiki/projects/sbca]] — created
