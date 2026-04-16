---
title: "Source: donormap repo"
type: source
tags: [source, globalfaces, firebase, snowflake, firetv, map]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: donormap repo

**Raw file:** `raw/repos/donormap`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

DonorMap is a real-time donor visualization tool for GlobalFaces Direct. React/Vite frontend with Firebase Cloud Functions pulling from Snowflake every 15 minutes into a Firestore cache. Frontend subscribes in real time. Domain-restricted to @globalfacesdirect.com. Includes a Fire TV sideloaded app for office ambient displays (TV3 and TV4 documented with IPs).

Not related to the `deployment` repo — DonorMap runs entirely on Firebase Cloud Functions; the `deployment` repo is a separate FastAPI backend for the Clarity Android app.

## Key Facts / Data Points

- Snowflake account: `su53882.canada-central.azure`; private key auth (`snowflake_key.p8`)
- GFD1 office excluded from live donor queries (documented in codebase)
- Fire TV office IPs: TV3=10.1.30.40, TV4=10.1.30.42; sideloaded via ADB
- ADB `pm clear` command documented for resetting TV pairing sessions
- Admin access: ctora@globalfacesdirect.com only (charity color customization)

## Pages Updated

- [[wiki/projects/donormap]] — created
