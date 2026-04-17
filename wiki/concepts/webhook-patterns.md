---
title: "Webhook Patterns"
type: concept
tags: [concept, webhooks, http, security, idempotency]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Webhook Patterns

A webhook is an inbound HTTP POST from an external service to your server, triggered by an event on their side. The server must respond quickly (usually within 5–10 seconds) or the sender retries.

## How It Is Used Here

Three projects receive webhooks:

| Project | Sender | Endpoint | Event |
|---------|--------|----------|-------|
| [[wiki/projects/clarity\|Clarity]] | Twilio | `POST /sms/reply` | Donor SMS YES/NO reply |
| [[wiki/projects/clarity\|Clarity]] | Stripe | `POST /webhook/stripe` | Payment lifecycle events |
| [[wiki/projects/sbca\|SBCA]] | Stripe | Stripe webhook handler | Dues payment confirmation |
| [[wiki/projects/haymaker\|Haymaker]] | Health Auto Export | `POST /api/webhooks/apple-health` | Apple Health data push |

## Key Properties / Mechanics

### Signature Validation

Every production webhook should be validated before processing. Senders sign the payload with a shared secret; the receiver re-computes the signature and rejects mismatches.

| Sender | Method |
|--------|--------|
| Twilio | `RequestValidator(TWILIO_AUTH_TOKEN)` — validates `X-Twilio-Signature` header against the full URL + POST params |
| Stripe | `stripe.webhooks.constructEvent(payload, sig, secret)` — validates `Stripe-Signature` header |
| Apple Health (Health Auto Export) | `X-API-Key` header check — no HMAC; OAuth browser flow not available from iOS background apps |

Clarity validates Twilio signature on every inbound SMS webhook. Without this, any HTTP client could forge a "YES" reply and trigger payment.

### Idempotency

Webhook senders retry on network failures. The receiver must de-duplicate to avoid double-processing.

**Pattern:** check `event_id` against a seen-events store before processing. Clarity's `EVENT_LOG` table in Snowflake uses `event_id` lookups for Stripe webhook de-duplication — inserts are append-only so a duplicate `event_id` is detectable.

### Respond Fast, Process Async

Stripe and Twilio expect a `200 OK` within seconds. Long processing (database writes, downstream API calls) should happen after responding, or in a background thread/queue. Clarity's `/sms/reply` handler updates `VERIFICATION_SMS` and returns immediately.

### Public Endpoint Requirement

Webhooks arrive from the internet — the endpoint must be publicly reachable without OAuth. Haymaker's `/api/webhooks/apple-health` uses `X-API-Key` auth specifically because Health Auto Export cannot complete an OAuth browser flow from a background iOS context.

## Relationships to Other Concepts

- [[wiki/concepts/stripe-payment-flows]] — Stripe webhooks confirm payment success; server-side verification depends on them
- [[wiki/concepts/oauth2-proxy-pattern]] — webhook endpoints must be exempted from OAuth proxy protection

## External References

- Twilio request validation: https://www.twilio.com/docs/usage/security
- Stripe webhook signatures: https://stripe.com/docs/webhooks/signatures
