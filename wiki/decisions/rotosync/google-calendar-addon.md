---
title: "Decision: Google Calendar Add-on for native conferencing integration"
type: decision
tags: [decision, adr, google-calendar, apps-script, integration]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Google Calendar Add-on for native conferencing integration

**Date:** 2026-04-17 (reconstructed from repo)
**Status:** `accepted`

## Context

Rotosync needed to integrate with users' meeting scheduling workflow. The question was whether to be a standalone app users visit separately, or to embed directly in Google Calendar as a conferencing option.

## Options Considered

1. **Standalone app** — users create Rotosync meetings separately, outside Calendar. Requires a separate "create meeting" form; attendee list must be re-entered.
2. **Calendar API polling** — poll Google Calendar for upcoming events and create rooms automatically. Requires OAuth consent per user; polling introduces latency.
3. **Google Calendar Add-on (Apps Script)** — registers Rotosync as a native conferencing provider. Users see "Rotosync" alongside Google Meet and Zoom when adding conferencing.

## Decision

Google Calendar Add-on via Apps Script. Key reasons:

- **Embedded in the workflow:** Users create meetings where they already work; no separate UI.
- **Attendee roster automatic:** Calendar attendees become the expected participant list in Firestore; the webapp filters access and carryover items based on the invite list.
- **Calendar event as source of truth:** Meeting title, time, organizer, and attendees come from the calendar event — no duplicate entry.
- **Bidirectional sync:** When a user moves an event or adds attendees, Apps Script triggers `updateConference` automatically — Daily.co room and Firestore stay in sync without user action.

## Consequences

- Google Workspace-only: Apps Script integration works only for `@globalfacesdirect.com` domain users; external guests cannot trigger room creation.
- Apps Script has execution quotas (6 min runtime limit, daily trigger limits).
- Edge cases: recurring events, events not yet saved, events with past dates — all require explicit handling in the Apps Script callbacks.

## Related Pages

- [[wiki/projects/rotosync]]
- [[wiki/decisions/rotosync/firebase-unified-backend]]

## Sources

- `raw/repos/rotosync` — docs/google-calendar-integration.md, functions/src/createConference.ts, functions/src/updateConference.ts, PLAN.md
