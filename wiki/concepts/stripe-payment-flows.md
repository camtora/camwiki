---
title: "Stripe Payment Flows"
type: concept
tags: [concept, stripe, payments, nfc, subscriptions]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Stripe Payment Flows

Stripe offers multiple payment integration patterns depending on context: card-not-present (e-commerce), card-present (in-person terminal), and recurring subscriptions. Two distinct patterns are used across this wiki's projects.

## How It Is Used Here

| Project | Pattern | Context |
|---------|---------|---------|
| [[wiki/projects/clarity\|Clarity]] | Stripe Terminal — Tap to Pay on Android | Door-to-door fundraising; card-present NFC tap |
| [[wiki/projects/sbca\|SBCA]] | CardField + PaymentIntent (server-side verification) | Mobile app; annual dues payment |

## Key Properties / Mechanics

### Pattern 1 — Stripe Terminal (Tap to Pay on Android)

Used when the payer is physically present and taps their card on the device.

Flow:
1. Backend issues a **connection token** (`POST /terminal/connection_token`) — scopes the Terminal session
2. App's `TapToPayDiscoveryConfiguration` uses the device's NFC chip as the reader (no external hardware)
3. Backend creates a **PaymentIntent** with `payment_method_types: ["card_present", "interac_present"]` + session/donor metadata
4. `collectPaymentMethod()` — donor taps card
5. `confirmPaymentIntent()` — charges card

**Monthly donations:** after tap, the generated card ID is attached to a Stripe `Customer` and a `Subscription` is created with `cancel_at` set 50 years forward. `AllowRedisplay.ALWAYS` must be set on `collectPaymentMethod` to enable card reuse.

**PCI scope:** the app never sees raw card data — Stripe Terminal tokenises everything.

### Pattern 2 — CardField + Server-side PaymentIntent Verification

Used for mobile e-commerce where the payer enters card details in-app.

Flow:
1. Client collects card details via Stripe's `CardField` component
2. Client creates a PaymentMethod and sends `paymentMethodId` to the server
3. Server creates a **PaymentIntent** with the member's ID bound in `metadata`
4. Client confirms the PaymentIntent
5. **Server re-fetches** the PaymentIntent and verifies:
   - `pi.status === 'succeeded'`
   - `pi.metadata.memberId === req.member.id`

The server-side re-fetch (step 5) is critical — it prevents a client from substituting a PaymentIntent from a different account or a lower amount. Documented in SBCA commit `7f4fe72`.

### Webhooks

Both patterns emit Stripe webhook events (`payment_intent.succeeded`, `customer.subscription.created`, etc.). Clarity logs these to `EVENT_LOG` for compliance audit. SBCA uses them for membership status updates.

See [[wiki/concepts/webhook-patterns]] for signature validation.

## Relationships to Other Concepts

- [[wiki/concepts/webhook-patterns]] — Stripe sends payment lifecycle events as webhooks
- [[wiki/concepts/snowflake]] — Clarity stores Stripe metadata and EVENT_LOG in Snowflake

## External References

- Stripe Terminal: https://stripe.com/docs/terminal
- Tap to Pay on Android: https://stripe.com/docs/terminal/payments/setup-reader/tap-to-pay-android
