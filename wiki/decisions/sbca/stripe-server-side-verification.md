---
title: "Decision: Stripe payment verified server-side before membership activation"
type: decision
tags: [decision, adr, stripe, payments, security]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Stripe payment verified server-side before membership activation

**Date:** 2026-04-16 (reconstructed; commit 7f4fe72 "fix: verify Stripe payment server-side")
**Status:** `accepted`

## Context

SBCA members pay annual dues via Stripe. The mobile app uses Stripe's `CardField` component to collect payment. The question was whether the client or server should be authoritative on whether payment succeeded before activating membership.

## Options Considered

1. **Client confirms, tells server to activate** — app receives Stripe's client-side confirmation, POSTs "payment succeeded" to backend. Simple but vulnerable: a crafted request can activate membership without payment.
2. **Webhooks only** — backend listens for Stripe `payment_intent.succeeded` webhook. No synchronous verification path; activation could be delayed or missed.
3. **Server creates PaymentIntent + server verifies status** — backend creates the PaymentIntent, client collects and confirms the card, backend then independently retrieves the PaymentIntent from Stripe to verify status and metadata before activating.

## Decision

Server-side verification. Flow:
1. Client calls backend to create a `PaymentIntent` (backend stores `memberId` in PI metadata)
2. App uses `CardField` to collect card details and confirms the PI client-side
3. App sends `paymentIntentId` to backend
4. Backend calls `stripe.paymentIntents.retrieve(paymentIntentId)`
5. Backend checks `pi.status !== 'succeeded'` → 402 if not paid
6. Backend checks `pi.metadata.memberId !== req.member.id` → 403 if metadata mismatch
7. Only then: membership activated

The metadata binding (step 6) prevents an attacker from reusing a valid PaymentIntent from a different member's transaction.

## Consequences

- Payment activation is synchronous and verified — no race condition between webhook and activation.
- `CardField` only — no `PaymentSheet` or other client-side flows that bypass server creation.
- Webhooks used for reconciliation and audit, not as the activation trigger.
- One additional Stripe API call per payment (the retrieve); negligible latency for an annual transaction.

## Related Pages

- [[wiki/projects/sbca]]

## Sources

- `raw/repos/sbca` — backend/src/routes/members.ts (lines 66–91), commit 7f4fe72
