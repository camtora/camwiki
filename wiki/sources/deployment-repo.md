---
title: "Source: deployment repo"
type: source
tags: [source, globalfaces, fastapi, stripe, twilio, snowflake]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: deployment repo

**Raw file:** `raw/repos/deployment`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

FastAPI Python backend for the Clarity Android app (GlobalFaces Direct). Single-file app (`main.py`) handling Stripe payments, Twilio SMS verification, Snowflake donor data, and fundraiser auth. Not related to DonorMap despite both being GlobalFaces projects — different database (`PHOENIX_APP_DEV` vs Fivetran), different hosting, different purpose.

## Key Facts / Data Points

- Minimal repo: `main.py` + `requirements.txt` only
- Snowflake: `PHOENIX_APP_DEV.CORE` schema; key-pair auth; Canada Central Azure
- Stripe: terminal connection tokens, PaymentIntents, SetupIntents, subscriptions, webhooks
- Twilio: SMS send/verify + inbound webhook
- Donor signatures uploaded to Snowflake internal stage `ASSETS_INT`

## Pages Updated

- [[wiki/projects/clarity]] — deployment backend documented there (combined page)
