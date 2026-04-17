---
title: "Decision: Self-hosted infrastructure, not cloud (Supabase/Vercel)"
type: decision
tags: [decision, adr, infrastructure, privacy, data-sovereignty]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Self-hosted infrastructure, not cloud (Supabase/Vercel)

**Date:** 2026-04-16 (reconstructed; documented in stack-comparison.md)
**Status:** `accepted`

## Context

Haymaker stores personal health data (weight, body composition, sleep, reproductive health, journal entries) and personal financial data (income, debts, bank transactions, spending). The question was whether to use managed cloud services (Supabase for database/auth, Vercel for hosting) or self-host everything on the home server.

A companion family app with similar tech uses Supabase + Vercel and reaches the right answer for that use case.

## Options Considered

1. **Supabase + Vercel** — Supabase wraps PostgreSQL with real-time subscriptions, Row-Level Security, connection pooling, and a management UI. Vercel auto-deploys from GitHub with zero infrastructure config. Standard managed stack.
2. **Self-hosted Docker Compose on home server** — PostgreSQL 16 in Docker, raw SQLAlchemy + Alembic migrations, MinIO for object storage, FastAPI container. All data stays on user-controlled hardware.

## Decision

Self-hosted Docker Compose. The driving constraint is data sovereignty: personal financial and health data should not reside on a third-party cloud provider's infrastructure. This is intentional design, not a cost or technical optimization.

From `stack-comparison.md`: "Haymaker runs on a personal server deliberately — it holds personal financial and health data, and keeping it off cloud hosting is a feature."

Specific data concerns:
- Journal entries (daily personal reflection)
- Reproductive/sex health tracking
- Bank transactions, debts, credit card balances
- Body composition measurements (body fat %, WHR, measurements)
- Progress photos (stored in self-hosted MinIO)

## Consequences

- **PostgreSQL** managed manually: migrations via Alembic, backups via `pg_dump` cron at 3 AM (7-day retention), connection pooling via SQLAlchemy connection pool.
- **MinIO** provides S3-compatible API for photos; no cloud vendor dependency; data stays on `HOMENAS`.
- **OAuth2 Proxy allowlist** instead of NextAuth — small known user group; manual approval gate is appropriate and keeps identity management simple.
- Not reachable from external networks without VPN or port forwarding (intentional; Haymaker is protected by OAuth2 Proxy).
- Supabase's RLS, real-time, and managed UI are foregone — acceptable trade-off for data sovereignty.
- Alembic migration discipline required; no automatic schema management.

## Related Pages

- [[wiki/projects/haymaker]]
- [[wiki/decisions/haymaker/fastapi-over-nextjs-api-routes]]
- [[wiki/infrastructure/oauth2-proxy]]

## Sources

- `raw/repos/haymaker/stack-comparison.md` — explicit rationale for self-hosted vs Supabase/Vercel
- `raw/repos/haymaker` — docker-compose.yaml (PostgreSQL, MinIO, backup scripts)
