---
title: "Source: clarity repo"
type: source
tags: [source, globalfaces, android, kotlin, stripe, twilio]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: clarity repo

**Raw file:** `raw/repos/clarity`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

Clarity is an Android app (Kotlin/Gradle) for GlobalFaces Direct door-to-door fundraising. Handles fundraiser login, donor sign-up, Stripe tap-to-pay and recurring subscriptions, and Twilio SMS verification. Paired with a FastAPI backend in the `deployment` repo.

## Key Facts / Data Points

- Android only (no iOS); Kotlin + Gradle build
- Stripe subscriptions noted as Canada-only (Stripe account is Canadian)
- Snowflake database: `PHOENIX_APP_DEV.CORE` (distinct from DonorMap's Fivetran database)
- Gamification features in active development (most recent commits)
- Backend in `deployment` repo (FastAPI, Python) — `main.py` + `requirements.txt` only, minimal repo

## Pages Updated

- [[wiki/projects/clarity]] — created (covers both clarity + deployment repos)
