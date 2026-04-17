---
title: "Decision: Email-based carryover item scoping with alias normalisation"
type: decision
tags: [decision, adr, firestore, carryover, email]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Email-based carryover item scoping with alias normalisation

**Date:** 2026-04-17 (reconstructed from repo)
**Status:** `accepted`

## Context

Blockers and action items created in one meeting must carry forward to future relevant meetings. The system needs a reliable key for matching items to the right people and meeting contexts across days and meeting types.

## Options Considered

1. **Meeting ID scoping** — items belong to a specific meeting instance. Simple but items don't carry forward; each meeting starts fresh.
2. **User ID scoping** — items belong to a user. Items appear in all of a user's meetings regardless of type — too broad.
3. **Email-based scoping with type rules** — items store attendee/assignee emails; visibility determined by email membership in the current meeting's attendee list, filtered by meeting type rules.

## Decision

Email-based scoping with `normalizeEmail()` alias support and per-type visibility rules.

- **Carryover key:** assignee/owner email addresses on each item document.
- **Alias normalisation:** `cameron.tora@globalfacesdirect.com` and `ctora@globalfacesdirect.com` refer to the same person. `normalizeEmail()` in `emailAliases.ts` maps variants to a canonical form before comparison.
- **1:1 isolation:** Items created in a 1:1 store `oneOnOneAttendees` (sorted pair of emails). Items from Alice+Bob's 1:1 never appear in Alice+Carol's 1:1.
- **Visibility enforced in two places:** webapp (`useCarryover.ts`) and Cloud Functions (`summarizeStandup.ts`) — both apply the same rules to prevent leakage.
- **Backfill migrations:** New fields (e.g., `oneOnOneAttendees`) are added via migration scripts (`backfillOneOnOneAttendees.ts`) to existing documents.

## Consequences

- If a person's email address changes (e.g., company rename), alias mappings must be updated in `emailAliases.ts`.
- Visibility logic lives in two places and must be kept in sync.
- Firestore schema is flexible (permissive rules allow document evolution) — migration scripts handle field backfills without downtime.

## Related Pages

- [[wiki/projects/rotosync]]
- [[wiki/decisions/rotosync/multi-meeting-type-architecture]]

## Sources

- `raw/repos/rotosync` — webapp/src/utils/emailAliases.ts, functions/src/backfillOneOnOneAttendees.ts, CLAUDE.md (Carryover Visibility Matrix, Email Alias Support sections)
