---
title: "Rotosync"
type: project
tags: [project, standup, meetings, firebase, daily, transcription, ai]
created: 2026-04-16
updated: 2026-04-17
source_count: 2
---

# Rotosync

Standup facilitation webapp with video conferencing, live transcription, and AI-powered meeting summaries. Integrates with Google Calendar as a conferencing option. Hosted on Firebase at `rotosync-7b249.web.app`.

## Status

Current status: `active`
Last meaningful change: 2026-04-16 (transcription warning indicator)

## Architecture

| Component | Tech |
|-----------|------|
| Frontend | React (Vite + TypeScript) |
| Backend | Firebase Cloud Functions |
| Database | Firestore |
| Auth | Google SSO (Firebase Auth) |
| Video/audio | Daily.co (WebRTC) |
| Transcription | Daily.co live transcript with speaker attribution |
| AI summaries | Google Gemini (Vertex AI) — switched from Claude |
| Email | Gmail API (domain-wide delegation) |
| Notifications | Slack webhook |
| Hosting | Firebase Hosting |
| Calendar | Google Calendar add-on |
| Project tracking | Monday.com API |

AI summarization switched from Claude to Gemini/Vertex AI (commit `95a6658`).

## Meeting Types

### Daily Standup
Team rotation (least-recently-first), round-table speaker turns, live transcript. AI summary at end produces: **synopsis**, **workstreams** (by project with green/yellow/red status), **action items** (with assignees and dates), **blockers**, **key decisions**, **per-person updates**. Carryover system persists open blockers and action items across standups.

Meeting type detected from calendar event title keywords: "standup", "stand-up", "daily", "sync" (without project-specific keywords).

### 1:1 Meeting
Private two-person meeting with a **persistent document** (`oneOnOneDocuments/{sortedEmails}`) that carries over between meetings with the same pair. Real-time AI extraction every 1 minute: extracts blockers, action items, updates rolling summary. At end, document is organized into Last Meeting Summary, Discussion Topics, and Workstreams sections. Shows previous 1:1 summary on join for context.

### Project Meeting
Focused discussion with a **persistent project document** (`projectDocuments/{projectName}`). Has a Purpose field (helps AI understand context), Timeframe, Google Drive links, and content area. AI updates the document during the meeting; at end, document is cleaned up and a meeting summary is added.

### Ad-hoc
Custom attendee selection, quick unstructured meetings.

## Core Features

**During meetings:**
- Daily.co WebRTC rooms (audio-only by default), screen sharing
- Live transcription with per-speaker attribution
- Real-time AI extraction — per-speaker summaries generated as each person finishes talking
- Active speaker detection (visual gold highlight on current turn)
- Raise hand — animated indicator for questions
- Meeting reactions (Zoom/Meet style emoji reactions)
- Microphone and camera device selection

**Standup-specific:**
- Team rotation (least-recently-first ordering)
- Round-table speaker turns
- Carryover system: blockers and action items persist across standups until resolved

**Wrap-up sections:**
- **Data Tickets** — Monday.com unassigned tickets with "Take it" assignment
- **CAB Board** — charity launches from Monday.com with tech status pills, D2D status
- **Open Forum** — review meeting blockers and action items

**Post-meeting:**
- Email summary to host (Gmail API, domain-wide delegation)
- Slack summary to webhook
- Summaries include: workstreams, action items, blockers

## Google Calendar Integration

Rotosync appears as a conferencing option on calendar events via a Google Workspace Apps Script add-on (`meet-addon/`). When a user selects "Rotosync" as conferencing, Apps Script calls the `createConference` Cloud Function → creates a Daily.co room → stores meeting data in Firestore → event gets join link `https://rotosync-7b249.web.app?room={conferenceId}`.

**Access control**: only emails on the calendar invite can join; host-only controls (Settings, History, Reports); teams and carryover items filtered to invited attendees.

**Countdown sync**: the 2-minute pre-meeting countdown is synced to the scheduled time, not to when you joined. Join 90s late → countdown shows 30s remaining. Join 2+ minutes late → standup already active.

**Moving a meeting**: `onCalendarEventUpdated` Apps Script trigger fires → calls `updateConference` Cloud Function → updates Firestore and Daily.co room. Next join sees updated schedule.

## Monday.com Integration

Two boards integrated into standup wrap-up sections:

| Board | Board ID | Wrap-up Section | Purpose |
|-------|----------|-----------------|---------|
| Data Tickets | `7475653020` | "Data Tickets" | Shows unassigned tickets; "Take it" dropdown assigns to team member; Monday.com automation moves ticket to "Open" group |
| Horizon Board 2.0 | `7464044973` | "Launches" | Upcoming charity launches grouped by week; editable Tech Status (Go/No Go/Conditional Go/At Risk) and Launch Notes; filtered: Launch Date ≥ today, not empty, status ≠ Cancelled |

Cloud Functions: `getMondayTickets`, `getMondayUsers`, `assignMondayTicket`, `getCharityLaunches`, `updateLaunchNotes`, `updateLaunchStatus`. Secret: `MONDAY_API_KEY` in Firebase Secrets.

## Infrastructure

- Hosted on Firebase (no home server dependency)
- `firebase.json`, `firestore.rules`, `firestore.indexes.json` in repo root
- Standalone transcript files in repo root (standup history from Jan 2026)

## Change Log

- 2026-04-16: Transcription warning indicator; fixed ad-hoc room transcription
- 2026-04-09: Switched AI from Claude to Gemini/Vertex AI
- 2026-01-21: Meeting reactions; 1:1 name display; attendance chart fix
- 2026-01-16: Microphone/camera device selection
- 2026-01-14: Client-side Firebase callable timeouts

## Key Decisions

- [[wiki/decisions/rotosync/firebase-unified-backend|Firebase as Unified Backend]] — Firestore real-time listeners, Cloud Functions, Auth, Hosting; beats SQL for multi-user video-enabled product.
- [[wiki/decisions/rotosync/daily-co-webrtc|Daily.co for WebRTC and Transcription]] — Built-in live transcription + active speaker events; audio-only default for standups.
- [[wiki/decisions/rotosync/google-calendar-addon|Google Calendar Add-on]] — Native conferencing provider; attendee roster and meeting metadata flow from calendar automatically.
- [[wiki/decisions/rotosync/multi-meeting-type-architecture|Multi-meeting-type Architecture]] — Title-pattern detection (standup/project/1:1); default-to-1:1; per-type carryover visibility matrix.
- [[wiki/decisions/rotosync/gemini-over-claude|Gemini over Claude for AI Extraction]] — Vertex AI ADC eliminates API key secret; same GCP project.
- [[wiki/decisions/rotosync/email-scoped-carryover|Email-scoped Carryover Items]] — Email-based matching with alias normalisation; 1:1 isolation via oneOnOneAttendees field.

## Known Issues

**Critical**
- **Google Meet transcript not implemented** — the Google Calendar add-on code path for fetching a Google Meet transcript returns an empty string. The feature appears in the UI but produces no output for meetings hosted on Google Meet (as opposed to Daily.co rooms). (`meet-addon/` Cloud Function)

**High**
- **Email summary not implemented** — post-meeting email dispatch calls `console.log` instead of sending via the Gmail API. Hosts do not receive summaries; no error is surfaced.
- **Daily.co transcription has no fallback** — if Daily.co live transcription fails or is unavailable, the AI summary pipeline receives an empty transcript. The meeting ends with no workstream or action item extraction, silently.

**Medium**
- **Gemini response parsing is fragile** — AI extraction parses Gemini's output via manual JSON regex extraction rather than a schema-validated response format. Unusual Gemini outputs (extra commentary, markdown fences) cause silent parse failures and missing action items.
- **Hardcoded wrap-up section owner emails** — the Data Tickets and Launches wrap-up sections check for specific email addresses (`strikha@`, `ctora@`) hardcoded in Cloud Function logic. Org changes require code deploys.
- **Browser cache requires hard refresh after deploy** — Firebase Hosting does not cache-bust the main JS bundle on deploy. Users on stale cache may run old code until they force-refresh.

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/rotosync-repo]] — meeting types, AI pipeline, Monday.com integration, Firebase architecture
