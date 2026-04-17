---
title: "Decision: Stripe Tap to Pay on Android (no external card reader)"
type: decision
tags: [decision, adr, stripe, payments, nfc]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Stripe Tap to Pay on Android (no external card reader)

**Date:** 2026-04-16 (reconstructed; commit 000e9e8 "Tap working with stripe in simulation")
**Status:** `accepted`

## Context

Clarity processes card payments in the field — donors tap their card at a door. The question was how to handle card-present payments without a physical card reader that fundraisers must carry and manage.

## Options Considered

1. **External Bluetooth card reader (Stripe Reader M2 / BBPOS)** — dedicated hardware, very reliable. But: hardware procurement cost, logistics (charging, pairing, replacing lost/broken units), setup friction at each door ("let me find my reader").
2. **Web-based checkout link** — send donor a link, donor pays on their own phone. Breaks the in-person flow; requires donor to have their phone out and internet access.
3. **Stripe Tap to Pay on Android** — uses the fundraiser tablet's built-in NFC chip as the card reader. No external hardware. Donor taps their card directly on the tablet.

## Decision

Stripe Terminal SDK's Tap to Pay on Android discovery configuration (`TapToPayDiscoveryConfiguration`). The phone's NFC chip becomes the card reader.

Flow:
1. Backend returns a Stripe Terminal connection token (`/terminal/connection_token`)
2. `TapToPayController.initializeTapToPay()` discovers the NFC reader (the phone itself) via `DiscoveryConfiguration.TapToPayDiscoveryConfiguration(isSimulated = false)`
3. Backend creates a `PaymentIntent` with `payment_method_types: ["card_present", "interac_present"]` and donor/session metadata
4. `collectPaymentMethod()` — donor taps card on tablet NFC
5. `confirmPaymentIntent()` — charges card or initiates subscription

For **monthly donations**: after tap, the generated card ID is retrieved, attached to a Stripe `Customer`, and a `Subscription` created with `cancel_at` set 50 years forward (effectively permanent until cancelled).

The app never sees raw card data — Stripe Terminal tokenizes everything. PCI compliance is fully delegated to Stripe.

## Consequences

- Zero hardware cost beyond the tablet itself; no reader logistics.
- `AllowRedisplay.ALWAYS` set on `collectPaymentMethod` to enable card reuse for subscriptions.
- NFC reliability varies by Android device and card type; field testing required per device model.
- Stripe Terminal requires Location permission (Stripe SDK requirement, not Tap to Pay itself) — handled in `MainActivity.kt`.
- Backend creates PaymentIntent with metadata (`session_id`, `donor_id`) for webhook enrichment and audit.

## Related Pages

- [[wiki/projects/clarity]]
- [[wiki/decisions/clarity/snowflake-as-database]]

## Sources

- `raw/repos/clarity` — TapToPayController.kt (full payment flow), build.gradle.kts (stripe.terminal libs), AndroidManifest.xml (NFC feature)
- `raw/repos/deployment` — main.py (/terminal/connection_token, /terminal/payment_intent, /subscriptions/create endpoints)
