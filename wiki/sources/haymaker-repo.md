---
title: "Source: haymaker repo"
type: source
tags: [source, health, habits, budget, fastapi, nextjs, postgresql]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: haymaker repo

**Raw file:** `raw/repos/haymaker`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

Haymaker is the largest personal project — a full-stack health, habit, and finance tracker. FastAPI Python backend + Next.js 15 frontend + PostgreSQL 16 + MinIO, self-hosted on Docker Compose. Features span daily check-ins, meal logging, workout tracking with Empower integration, a sophisticated budget module (pay period forecasting, credit card integration, debt payoff forecaster), and analytics.

Key integrations: Withings (pull — weight/body fat/muscle via OAuth2), Apple Health (push — via Health Auto Export webhook, capturing sleep stages and nutrition macros), Discord reminder crons (9am–10pm, only fire if task incomplete), daily backup scripts (pg_dump + rsync).

Multi-user auth was added via a registration request → Discord notification → admin approval flow. 93 user-scoped endpoints all use `validate_user_access`. Beat training restricted to user_id=1.

The repo also contains a `stack-comparison.md` comparing Haymaker's architecture to a hypothetical family app — useful context for architectural decisions (why FastAPI over Next.js API routes, why MinIO over cloud storage, why self-hosted).

## Key Facts / Data Points

- 93 `/users/{user_id}/...` API endpoints with `validate_user_access` dependency
- Apple Health integration: webhook at `POST /api/webhooks/apple-health`, requires Health Auto Export iOS app (~$5)
- Withings: OAuth2 flow in Settings; limitation — Apple Watch sleep synced via Withings not available in their API
- Discord reminders: 6 per day, cron-based, only fire if task incomplete
- Daily backup: `scripts/backup.sh`, 3 AM, 7-day retention on DB dumps
- Budget module: 4 debt payoff methods, pay-period forecasting, envelope spending, credit card bidirectional sync
- Meal planning: templates + weekly planning added in Phase 4/5 (2026 commits)
- Everyday Tracker: year-long habit heatmap visualization
- Stack comparison doc: `stack-comparison.md` — deliberate choices for FastAPI, MinIO, self-hosting

## Relevance to Wiki

Most detailed personal project — covers health, finance, and habit tracking. Architectural decisions well-documented in `stack-comparison.md`.

## Contradictions or Tensions

None — first ingest.

## Pages Updated

- [[wiki/projects/haymaker]] — created
