---
title: "Haymaker"
type: project
tags: [project, health, habits, budget, fastapi, nextjs, postgresql]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Haymaker

Personal health, habit, and finance tracking application. The largest and most feature-rich personal project — covers daily physical check-ins, meal logging, workout tracking, budget/cashflow management, and goal setting. Deliberately self-hosted to keep sensitive health and financial data off third-party cloud.

## Status

Current status: `active`
Last meaningful change: 2026-04-16 (expected/actual balance display + adjustment modal)

## Architecture

```
Internet → nginx-proxy → OAuth2 Proxy → Next.js frontend (port 3000)
                                              ↓ API calls
                                        FastAPI backend (port 8000)
                                              ↓
                                        PostgreSQL 16 + MinIO
```

| Component | Tech | Notes |
|-----------|------|-------|
| Frontend | Next.js 15, React, Tailwind CSS | App Router, `apps/web/` |
| Backend | FastAPI (Python 3.12) | Compute-heavy logic warrants separate service |
| Database | PostgreSQL 16 + SQLAlchemy + Alembic | Fine-grained ORM control for complex queries |
| File storage | MinIO (S3-compatible) | Progress photos, attachments — self-hosted |
| Auth | OAuth2 Proxy + custom `validate_user_access` | Small known user group; manual approval gate |
| Container | Docker Compose | `docker-compose.yaml` at repo root |

FastAPI was chosen over Next.js API routes because the backend does heavy computation: budget forecasting across pay periods, nutrition and macro tracking, workout scoring, cashflow modelling. Python's ecosystem is well-suited to this.

## Multi-User Auth

Haymaker started single-user and was refactored to multi-user. Registration flow:

1. Visitor submits registration form on camerontora.ca
2. Webhook creates `RegistrationRequest` in Haymaker DB (status: pending)
3. Discord notification sent to admin channel
4. Admin reviews at `/admin` — approves or denies
5. On approve: admin adds email to OAuth2 Proxy allowlist + sends user the link
6. User signs in with Google → account auto-created on first request via `x-forwarded-email` header

93 `/users/{user_id}/...` API endpoints use `validate_user_access` dependency — users can only access their own data. Beat training endpoints (5 routes) restricted to `user_id=1` only.

## Features

### Daily Check-In

Daily log of weight, body composition, sleep quality, macros, and journal reflection. Auto-fill available when Withings or Apple Health data is present. Accessible via mobile-optimized `/mobile` page.

### Habit Tracking

- Habit types: checkbox, timer, numeric target
- Habit categories with configurable `show_on_dashboard` and sort order
- Tracker habits: `counts_towards_completion` flag differentiates target habits from counters
- Habits can be linked to goals; exceeding targets is allowed after completion
- Everyday Tracker: year-long calendar heatmap with alphabetical habit selection
- Auto-complete habits: Weigh-In, Progress Pic, Take Measurements, workout sub-habits

### Meal Logging

- Log breakfast, lunch, dinner, snacks with macros (calories, protein, carbs, fat, fiber)
- Meal templates for repeated meals
- Weekly meal planning
- Per-meal calorie/macro targets configurable in settings
- Net carbs used in nutrition analytics (carbs minus fiber)

### Workout Tracking

- Integration with **Empower** fitness sessions (imported automatically)
- Auto-populates workout habits (situps, pushups, stairs, cardio) from workout entry data
- Retroactive editing of past workouts via date navigation
- Configurable cardio activity types with optional quantity fields (e.g., km, flights of stairs)
- Fitness Events: recurring calendar entries for classes, skating sessions, etc. with location management

### Budget & Cashflow

Most complex module. Covers:

- **Pay period tracking** — income, expenses, debts organized by pay period
- **2026 cashflow forecast** — projects forward across all pay periods
- **Scenario planning** — model what-if changes to income/expenses
- **Credit card integration** — CC bank accounts auto-create linked debt entries; CC fields sync bidirectionally (APR, minimum payment, balance, due day); balance auto-syncs after CSV import
- **Envelope spending** — categorized bank transactions, CC payments excluded via "Mark as Transfer"
- **Debt payoff forecaster** — four methods: Snowball, Avalanche, Highest Payment, Custom; lump sum payments (tax refunds, bonuses); side-by-side comparison; goal-based reverse calculator (target date → required payment); plan implementation links to recurring expense for cashflow
- **Expected/actual balance display** with adjustment modal

### Goals & Analytics

- Goal setting for weight, body fat, fitness with date-bound checkpoints
- Checkpoint cards show muscle %, body fat %, WHR targets with color-coded delta
- Analytics pages: body trends, nutrition charts, habit completion, workout history
- `/analytics/life` — AI-powered journal summary
- Recovery history: sleep times, scores, outlier filtering

### Integrations

**Withings** (pull model):
- OAuth2 flow in Settings → connect Withings account
- Auto-syncs weight, body fat, muscle mass
- "Pull from Withings" manual refresh available
- Limitation: Apple Watch sleep synced via Withings does not appear in their API

**Apple Health** (push model, via [Health Auto Export](https://www.healthyapps.dev/) iOS app ~$5):
- iPhone app sends webhooks to `POST /api/webhooks/apple-health`
- Captures: sleep (duration, score, REM/deep/core stages), nutrition (calories, protein, carbs, fat, fiber, sugar, micronutrients)
- Configured with `X-API-Key` generated in Haymaker Settings → Apple Health

**Discord reminders** (cron, only fires if task incomplete):

| Time | Reminder |
|------|----------|
| 9:00 AM | Weigh-in & progress photo |
| 10:00 AM | Log breakfast |
| 2:00 PM | Log lunch |
| 8:00 PM | Log dinner |
| 9:00 PM | Complete daily habits |
| 10:00 PM | Daily check-in & journal |

### Backups

Daily at 3 AM:
- Database: `pg_dump` with 7-day retention
- Photos: `rsync` to backup location

```bash
./scripts/backup.sh
# Restore: gunzip -c backup.sql.gz | docker exec -i haymaker_db_1 psql -U haymaker -d haymaker
```

## Infrastructure

- URL: `https://haymaker.camerontora.ca` (OAuth2 protected)
- Docker network: `haymaker_default` (172.18.0.0/16)
- Has its own OAuth2 Proxy instance (separate from shared infrastructure proxy)
- Reverse proxy: [[wiki/infrastructure/nginx-reverse-proxy]]
- Auth: [[wiki/infrastructure/oauth2-proxy]]
- Monitoring: [[wiki/infrastructure/gcp-external-monitoring]] checks reachability

## Key Files

| Path | Purpose |
|------|---------|
| `apps/api/app/routes.py` | All 93 API endpoints |
| `apps/api/app/models.py` | SQLAlchemy models |
| `apps/api/app/deps.py` | Auth: `get_current_user`, `validate_user_access`, `get_admin_user` |
| `apps/api/alembic/` | Database migrations |
| `apps/web/src/app/` | Next.js pages: check-in, habits, meal-plan, budget, workouts, analytics, goals, settings, admin |
| `scripts/backup.sh` | Automated database + photo backup |
| `scripts/send-reminder.sh` | Discord reminder sender |

## User Settings (via `/settings`)

| Section | Configurable fields |
|---------|-------------------|
| Profile | Height, birthdate, sex |
| Timezone | For daily resets and reminder timing |
| Nutrition targets | Protein min/max (g/kg), max carbs %, max fat %, calorie target |
| Calorie calculation | Activity factor, kcal/kg, max deficit, min calorie target |
| BMR formula | Mifflin-St Jeor or custom coefficients |
| Daily fitness goals | Situps, pushups, stairs, cardio (minutes) |
| Integrations | Discord webhook URL, Withings OAuth, Apple Health API key |

## Open Questions / Backlog Ideas

- **Renpho integration** — body circumference measurements; API unlikely, Apple Health only exposes waist
- **Apple Health nutrition via MyFitnessPal** — MyFitnessPal API is private, not accepting new developers

## Change Log

- 2026-04-16: Expected/actual balance display + adjustment modal on budget page
- 2026-01-17: Withings integration (weight, body fat, muscle mass)
- 2026-01-14: Credit card integration, debt payoff forecaster, dashboard muscle tracking
- 2026-01-13: Multi-user auth, new user onboarding, habits/goals alignment, Apple Health integration

## Sources

- [[wiki/sources/haymaker-repo]] — full feature documentation, architecture decisions, API reference
