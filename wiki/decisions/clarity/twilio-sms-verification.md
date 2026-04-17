---
title: "Decision: Twilio SMS for donor verification (YES/NO consent)"
type: decision
tags: [decision, adr, twilio, sms, compliance, donor-verification]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Twilio SMS for donor verification (YES/NO consent)

**Date:** 2026-04-16 (reconstructed from deployment repo)
**Status:** `accepted`

## Context

Door-to-door fundraising carries a compliance risk: a fundraiser could enter incorrect donor information (wrong amount, wrong phone number, wrong charity) either accidentally or maliciously. Clarity needed a mechanism to confirm the donor's intent before charging their card.

## Options Considered

1. **In-app signature only** — donor signs on the tablet screen. Confirms physical presence but doesn't verify the details they agreed to.
2. **Email verification** — send confirmation to donor email. Slower (donor may not check immediately); doesn't work for donors without email or who don't remember their address.
3. **Twilio SMS YES/NO** — send a summary SMS to the donor's phone ("You're agreeing to $X/month to [Charity]. Reply YES to confirm, NO to cancel"). Donor replies from their own device. Creates an auditable consent trail.

## Decision

Twilio SMS verification as a mandatory step before payment collection. Flow:

1. Fundraiser enters donor's phone number
2. Backend (`/verification/sms/send`) formats a summary message ("Hi [Fundraiser], confirming [Donor Name] agrees to [gift type] $[amount] to [Charity]? YES/NO") and sends via Twilio
3. App polls `/verification/sms/status` every 2 seconds (`LaunchedEffect` loop in `SmsVerifyScreen.kt`)
4. Twilio receives the donor's reply and POSTs it to `/sms/reply` webhook
5. Backend validates Twilio signature, parses YES/NO/INVALID, updates `VERIFICATION_SMS` record
6. On YES: proceed to payment. On NO: return to form for corrections.

Twilio request signature is validated on every inbound webhook (`RequestValidator(TWILIO_AUTH_TOKEN)`) to prevent injection. SMS events are logged to `EVENT_LOG` (`SMS_SENT`, `SMS_REPLY_YES`, `SMS_REPLY_NO`) for compliance audit.

Messaging Service SID used when configured (allows geolocation number pool); falls back to single `TWILIO_FROM_NUMBER`.

## Consequences

- Adds 30–120 seconds of wait time per donation (donor must receive and reply to SMS). Accepted trade-off for compliance protection.
- Donors without mobile phones or SMS cannot proceed — edge case; fundraisers handle via supervisor escalation.
- `VERIFICATION_SMS` table preserves the exact message sent and reply received — full audit trail if a donor disputes a charge.
- SMS delivery SLA is Twilio's; international numbers require Twilio international SMS add-on.
- The 2-second polling loop in the app is a simple implementation; a WebSocket push would be more elegant but adds infrastructure for minimal gain given the human-speed interaction.

## Related Pages

- [[wiki/projects/clarity]]
- [[wiki/decisions/clarity/snowflake-as-database]]

## Sources

- `raw/repos/deployment` — main.py (send_verification_sms(), /sms/reply webhook, EVENT_LOG inserts), requirements.txt (twilio)
- `raw/repos/clarity` — SmsVerifyScreen.kt (2s polling loop)
