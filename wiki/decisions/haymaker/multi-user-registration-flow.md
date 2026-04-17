---
title: "Decision: Multi-user registration via Discord + manual OAuth2 allowlist"
type: decision
tags: [decision, adr, auth, multi-user, oauth2]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Multi-user registration via Discord + manual OAuth2 allowlist

**Date:** 2026-01-13 (from changelog; branch: feature/multi-user-auth)
**Status:** `accepted`

## Context

Haymaker started as a single-user app with `USER_ID=1` hardcoded throughout. The refactor to multi-user needed a registration and approval flow. The user group is small and known — household members or specific invited individuals — not public registration.

## Options Considered

1. **NextAuth + Google OAuth** — standard multi-user auth; users sign up themselves; no manual gate. Better for a public app where any Google account can register.
2. **Keep single-user** — no registration at all; simpler forever. Rejected because other household members wanted access.
3. **OAuth2 Proxy + manual allowlist + auto-create on first login** — maintain a curated `authenticated_emails.txt`; admin approves requests manually; account auto-created in DB on first authenticated request.

## Decision

OAuth2 Proxy allowlist with a structured approval workflow:

1. Visitor submits registration form on camerontora.ca (name + email)
2. Webhook creates `RegistrationRequest` in Haymaker DB (status: `pending`)
3. Discord notification sent to admin channel with requester details
4. Admin reviews at `/admin` — approves or denies
5. **On approve:** Admin manually adds email to `/etc/oauth2-proxy/authenticated_emails.txt` and sends user the link
6. User signs in with Google → OAuth2 Proxy allows them → `get_current_user()` auto-creates `User` record on first request using `x-forwarded-email` header

`stack-comparison.md` notes: "Small known user group; manual approval gate is appropriate." The overhead of managing a text file is acceptable; the benefit is a simple, auditable allowlist with no self-serve account creation.

**Data isolation:** All 93 `/users/{user_id}/...` endpoints use the `validate_user_access(user_id, current_user)` dependency — users can only read/write their own data. Beat training (5 routes) additionally locked to `user_id=1` via `validate_beat_training_access()` — custom feature not intended for other users.

**Existing data preserved:** User ID 1 is always `cameron.tora@gmail.com`; seed data migrations conditionally insert defaults only if `user_id=1` exists, so fresh installs are safe and existing installs keep their history.

## Consequences

- No self-serve registration; all new users require admin action (adding to allowlist file). Acceptable at this scale.
- `RegistrationRequest` table with Discord embed updates (embed updated on approve/deny) provides audit trail.
- Admin `/admin` page with filter tabs (All/Pending/Approved/Denied) + copy-email button for allowlist convenience.
- Discovery of an unauthorized user requires removing them from `authenticated_emails.txt` and reloading OAuth2 Proxy — no DB-level revocation needed.

## Related Pages

- [[wiki/projects/haymaker]]
- [[wiki/infrastructure/oauth2-proxy]]
- [[wiki/decisions/haymaker/self-hosted-not-cloud]]

## Sources

- `raw/repos/haymaker/docs/multi-user-refactor.md` — complete 339-line refactor specification
- `raw/repos/haymaker/stack-comparison.md` — rationale for OAuth2 Proxy over NextAuth
- `raw/repos/haymaker` — apps/api/app/deps.py (auth dependencies), apps/web/src/app/admin/page.tsx
