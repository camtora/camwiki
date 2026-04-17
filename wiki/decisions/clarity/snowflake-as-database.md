---
title: "Decision: Snowflake as transactional database, warehouse, and asset store"
type: decision
tags: [decision, adr, snowflake, database, compliance]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Snowflake as transactional database, warehouse, and asset store

**Date:** 2026-04-16 (reconstructed from deployment repo)
**Status:** `accepted`

## Context

GlobalFaces Direct already used Snowflake as a data warehouse (DonorMap reads from the same Snowflake instance). Clarity needed a database for donor records, sessions, payments, and signatures. The question was whether to introduce a separate transactional database (PostgreSQL) or use Snowflake for everything.

## Options Considered

1. **PostgreSQL + separate analytics pipeline** — standard OLTP DB; ETL to Snowflake for reporting. Two systems to maintain; ETL latency.
2. **MongoDB** — flexible schema; but no clean Pydantic integration; Snowflake's semi-structured JSON handling is comparable.
3. **Snowflake for everything** — transactions (`DONOR`, `FUNDRAISER`, `SESSION`, `PAYMENT`), event log (`EVENT_LOG`), and file assets (signatures in Snowflake internal stage). One system; compliance-friendly; existing org relationship.

## Decision

Snowflake for all persistence. Key reasons:

- **Compliance legacy**: GlobalFaces Direct is a compliance-heavy nonprofit. Snowflake provides immutable storage with time-travel — audit queries are first-class, not bolted on.
- **`EVENT_LOG` as immutable ledger**: All Stripe webhook events, SMS events, and app events are appended to `EVENT_LOG`. This table is never updated — only inserted. Webhook de-duplication uses `event_id` lookups.
- **Internal stages for signatures**: Donor signatures (PNG) stored in `@PHOENIX_APP_DEV.CORE.ASSETS_INT/signatures/`. `GET_PRESIGNED_URL()` generates 1-hour temporary HTTPS links for retrieval. No S3 account or bucket management needed.
- **Unified analytics**: `DONOR`, `SESSION`, `PAYMENT` data lives alongside the existing DonorMap analytics tables in the same Snowflake instance — cross-referencing is trivial.
- **Private key auth**: Backend connects via JWT/private key stored in Azure App Service environment variables (not password-based) — `SNOW_PRIVATE_KEY_CONTENT` env var on Azure, file path locally.

Schema highlights: `SESSION` (fundraiser session state), `VERIFICATION_SMS` (outbound SMS + inbound reply), `PAYMENT` (Stripe metadata, amounts), `PAYMENT_METHOD` (tokenized card reference).

New connection per request (no connection pool) — acceptable given Snowflake's connection overhead for the traffic volume.

## Consequences

- Snowflake is not optimized for high-frequency OLTP (no row-level locking, no sub-millisecond queries). Acceptable for fundraising workflows where transactions happen every few minutes, not per second.
- Connection per request adds ~200–500ms overhead per API call. Mitigated by Uvicorn async workers.
- `EVENT_LOG` provides a full audit trail of the payment lifecycle — required for nonprofit compliance.
- Rotating private key credentials requires updating Azure App Service env var and redeploying — no in-flight session disruption.

## Related Pages

- [[wiki/projects/clarity]]
- [[wiki/projects/donormap]]

## Sources

- `raw/repos/deployment` — main.py (get_snowflake_ctx(), EVENT_LOG inserts, upload_signature(), presign_stage_url()), requirements.txt
