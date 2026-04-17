---
title: "Snowflake"
type: concept
tags: [concept, snowflake, database, warehouse, analytics]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Snowflake

Snowflake is a cloud data platform optimised for analytical workloads (OLAP). It uses columnar storage, separates compute from storage, and provides features like time-travel (query historical data), zero-copy cloning, and semi-structured JSON handling. It is not designed for high-frequency transactional workloads (OLTP).

## How It Is Used Here

Snowflake is used by two GlobalFaces Direct projects, both on the same instance (`canada-central.azure`):

| Project | Database | Use |
|---------|----------|-----|
| [[wiki/projects/clarity\|Clarity]] | `PHOENIX_APP_DEV.CORE` | Transactional donor records, payments, event log, signature storage |
| [[wiki/projects/donormap\|DonorMap]] | Same instance | Analytics — FSA choropleth, donor visualisation |

Clarity's use of Snowflake as a **transactional database** is unconventional — see [[wiki/decisions/clarity/snowflake-as-database]] for the rationale (compliance legacy, unified analytics, no second system).

## Key Properties / Mechanics

### Tables Used by Clarity

| Table | Purpose |
|-------|---------|
| `DONOR` | Donor records |
| `FUNDRAISER` | Field rep records |
| `SESSION` | Fundraiser session state |
| `PAYMENT` | Stripe metadata, amounts |
| `PAYMENT_METHOD` | Tokenised card reference |
| `VERIFICATION_SMS` | Outbound SMS message + inbound reply |
| `EVENT_LOG` | Append-only audit ledger |

### EVENT_LOG as Immutable Ledger

`EVENT_LOG` is the compliance audit trail. It is **insert-only — never updated or deleted**. Every significant action appends a row:

| Event type | Trigger |
|------------|---------|
| `SMS_SENT` | Twilio SMS dispatched |
| `SMS_REPLY_YES` | Donor confirmed |
| `SMS_REPLY_NO` | Donor cancelled |
| Stripe webhook events | Payment lifecycle |

Webhook de-duplication uses `event_id` lookups against this table — before processing a Stripe event, Clarity checks whether that `event_id` already exists.

### Internal Stage for Signatures

Donor signatures (PNG) are stored in a Snowflake **internal stage** rather than an external object store (S3, Azure Blob):

```
@PHOENIX_APP_DEV.CORE.ASSETS_INT/signatures/
```

`GET_PRESIGNED_URL()` generates a 1-hour temporary HTTPS link for retrieval. No S3 account or bucket management required — Snowflake handles storage internally.

### Time-travel

Snowflake retains historical versions of data for a configurable period (default 1 day, up to 90 days on Enterprise). This means audit queries can reconstruct past states:

```sql
SELECT * FROM DONOR AT (TIMESTAMP => '2026-01-01 00:00:00');
```

For a compliance-heavy nonprofit, this is audit-query-first design, not bolted-on logging.

### Connection Model

Clarity's backend opens a **new Snowflake connection per request** — no connection pool. This adds ~200–500ms overhead per API call, mitigated by Uvicorn async workers. Snowflake connections are heavyweight (JWT handshake + session setup); connection pooling is complex and the traffic volume (one donation every few minutes) doesn't warrant it.

### Authentication

Private key authentication via JWT — not password-based:
- **Azure (production):** `SNOW_PRIVATE_KEY_CONTENT` environment variable in Azure App Service
- **Local dev:** file path to `snowflake_key.p8`

Rotating credentials requires updating the Azure App Service env var and redeploying — no in-flight session disruption since connections are per-request.

### OLAP Trade-offs in OLTP Use

| Limitation | Impact for Clarity |
|------------|-------------------|
| No row-level locking | Acceptable — donations happen every few minutes, not concurrently |
| No sub-millisecond queries | Acceptable — fundraising UX tolerates 200–500ms |
| Columnar storage overhead for small row inserts | Acceptable at low volume |

## Relationships to Other Concepts

- [[wiki/concepts/webhook-patterns]] — Stripe webhook events are de-duplicated via EVENT_LOG `event_id` lookups
- [[wiki/concepts/stripe-payment-flows]] — Stripe metadata stored in PAYMENT and EVENT_LOG tables

## External References

- Snowflake internal stages: https://docs.snowflake.com/en/user-guide/data-load-local-file-system-create-stage
- GET_PRESIGNED_URL: https://docs.snowflake.com/en/sql-reference/functions/get_presigned_url
