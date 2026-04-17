---
title: "Clarity"
type: project
tags: [project, globalfaces, android, kotlin, stripe, twilio]
created: 2026-04-16
updated: 2026-04-17
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

## Key Decisions

- [[wiki/decisions/clarity/android-only-platform|Android-only Platform]] — Stripe Tap to Pay on Android available first; enterprise tablets cheaper; NFC required feature.
- [[wiki/decisions/clarity/stripe-tap-to-pay|Stripe Tap to Pay on Android]] — Device NFC chip as card reader; no external hardware; monthly donations create Stripe Subscription.
- [[wiki/decisions/clarity/snowflake-as-database|Snowflake as Database]] — Compliance legacy; EVENT_LOG as immutable ledger; signatures in internal stage; time-travel audit queries.
- [[wiki/decisions/clarity/twilio-sms-verification|Twilio SMS Verification]] — Mandatory YES/NO consent before payment; 2s polling loop; Twilio signature validation on every inbound webhook.

## Known Issues

**Critical**
- **Duplicate endpoint at main.py line ~849** — a second definition of a route references undefined variables and would crash the backend process if the code path is ever hit. Identified via code inspection.
- **Twilio webhook signature validation disabled** — the inbound Twilio SMS handler has `validate_signature` commented out with a "re-enable later" note. Any HTTP client can POST fake SMS consent events.

**High**
- **CORS wildcard `allow_origins=["*"]`** — set in FastAPI middleware with a `# TODO: tighten` comment. Exposes the API to cross-origin requests from any domain.
- **No transaction rollback on multi-step payment flows** — if a donor record is created in Snowflake but the Stripe subscription call subsequently fails, the Snowflake `EVENT_LOG` row is orphaned with no corresponding Stripe object.
- **Session ownership not validated** — session endpoints do not verify that the requesting fundraiser owns the session_id in the URL, enabling cross-session data reads.

**Medium**
- **Fallback donor email `"donor@example.com"`** — used when no email is provided; creates real Stripe customer objects with a placeholder address that is hard to clean up later.
- **NFC hard failure with no retry** — tap-to-pay NFC failure shows an error but does not offer a retry flow; fundraiser must restart the payment flow manually.

## Open Questions

- Gamification scope unclear from commit messages alone

## Sources

- [[wiki/sources/clarity-repo]] — Android app structure, Stripe/Twilio setup
- [[wiki/sources/deployment-repo]] — FastAPI backend, Snowflake schema, endpoint reference
