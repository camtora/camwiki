---
title: "Decision: FastAPI (Python) over Next.js API routes"
type: decision
tags: [decision, adr, backend, fastapi, python]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: FastAPI (Python) over Next.js API routes

**Date:** 2026-04-16 (reconstructed; documented in stack-comparison.md)
**Status:** `accepted`

## Context

Haymaker is a monorepo with a Next.js frontend. The question was whether to co-locate API logic in Next.js API routes (the simpler default) or to run a separate Python backend service. A companion project (a family app with similar architecture) uses Next.js API routes and has no separate backend.

## Options Considered

1. **Next.js API routes** — API logic lives alongside the frontend in a single Next.js service. One container, one deploy, less infrastructure. Appropriate when the backend is primarily CRUD.
2. **FastAPI (Python, separate service)** — Dedicated backend container running Python. More infrastructure, but Python's data/science ecosystem is better suited to heavy computation.

## Decision

FastAPI with Python 3.12, running as a separate Docker service. The backend performs computation that would be painful in TypeScript:
- **Budget forecasting** — rolling pay-period cashflow projections, debt simulation across dozens of periods with compound interest
- **Macro/nutrition targets** — BMR calculation with multiple formula options (Mifflin-St Jeor, custom coefficients), activity factor adjustments, macro constraint solving
- **Workout habit scoring** — effort score from HR, calories, duration; multi-factor sleep score with quadratic penalty functions
- **Debt payoff forecaster** — binary search for goal-based payment calculation; four repayment strategies with cascade logic

These are not CRUD. Python's `zoneinfo`, `decimal`, and numeric operations are well-suited; TypeScript would add significant complexity with no benefit.

`stack-comparison.md` (repo root) explicitly states: "Haymaker's compute-heavy logic warrants a dedicated service. If the family app is primarily CRUD + Google Calendar sync + AI chat, Next.js API routes are sufficient — a separate Python service would be premature infrastructure."

## Consequences

- Two containers to maintain (`api` + `web`) instead of one.
- Next.js API routes proxy to FastAPI (`/api/users/me` → `http://api:8000/users/me`) to keep same-origin calls from the frontend — no CORS complexity.
- Python's SQLAlchemy + Alembic stack requires more migration discipline than Supabase would, but provides fine-grained ORM control for complex relational queries across budget periods and cashflow.
- 14,278-line `routes.py` covering all 93 endpoints — monolithic but well-understood.

## Related Pages

- [[wiki/projects/haymaker]]
- [[wiki/decisions/haymaker/self-hosted-not-cloud]]

## Sources

- `raw/repos/haymaker/stack-comparison.md` — explicit side-by-side with family app
- `raw/repos/haymaker` — docker-compose.yaml (api + web services), apps/api/app/routes.py
