---
title: "Clarity"
type: project
tags: [project, globalfaces, android, kotlin, stripe, twilio]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Clarity

Android app for GlobalFaces Direct door-to-door fundraising. Enables field fundraisers to sign up donors, process tap-to-pay and recurring Stripe payments, and verify donor identity via Twilio SMS. Paired with a FastAPI backend (`deployment` repo).

## Status

Current status: `active`
Last meaningful change: 2026-04-16 (gamification features)

## Architecture

| Component | Tech |
|-----------|------|
| Mobile app | Android (Kotlin, Gradle) |
| Backend | FastAPI (Python) — `deployment` repo |
| Payments | Stripe (tap-to-pay terminal + recurring subscriptions) |
| SMS | Twilio (donor identity verification) |
| Database | Snowflake (`PHOENIX_APP_DEV.CORE`) |
| Storage | Snowflake internal stage (`ASSETS_INT`) for donor signatures |

## Features

- **Fundraiser login** — field reps authenticate to access campaigns
- **Donor sign-up flow** — capture donor details, charity, campaign
- **Tap-to-pay** — Stripe terminal integration for one-time donations
- **Recurring subscriptions** — Stripe subscription creation with customer upsert
- **SMS verification** — Twilio SMS to verify donor identity; status polling
- **Donor signatures** — uploaded to Snowflake internal stage
- **Gamification** — recent commits indicate in-progress gamification features

## Backend (`deployment` repo)

FastAPI app (`main.py`) — key endpoints:

| Endpoint | Purpose |
|----------|---------|
| `POST /log-event` | Log donor/session events to Snowflake |
| `POST /verification/sms/send` | Send Twilio SMS verification |
| `GET /verification/sms/status` | Check SMS verification status |
| `POST /terminal/connection_token` | Stripe terminal connection token |
| `POST /terminal/payment_intent` | Create terminal payment intent |
| `POST /payment_intent` | Create standard payment intent |
| `POST /subscriptions/create` | Create recurring subscription |
| `POST /customer/upsert` | Create or update Stripe customer |
| `POST /webhook/stripe` | Handle Stripe webhook events |
| `POST /webhook/twilio` | Handle Twilio inbound SMS |
| `POST /donor/upsert` | Create or update donor record in Snowflake |
| `GET /donor/{donor_id}` | Fetch donor by ID |
| `GET /products/campaign/{campaign_id}` | Get campaign products |

Snowflake connection: `canada-central.azure` account, key-pair auth (`snowflake_key.p8`).

## Notes

- Stripe subscriptions only work in Canada (Stripe account is Canadian)
- `deployment` repo is the backend only — no frontend or infra config
- Related to [[wiki/projects/donormap]] (both GlobalFaces Direct) but independent systems — different databases, different hosting

## Change Log

- 2026-04-16: Gamification features (in progress)
- Earlier: Recurring subscriptions, tap-to-pay, Twilio SMS, donor signatures

## Open Questions

- Gamification scope unclear from commit messages alone

## Sources

- [[wiki/sources/clarity-repo]] — Android app structure, Stripe/Twilio setup
- [[wiki/sources/deployment-repo]] — FastAPI backend, Snowflake schema, endpoint reference
