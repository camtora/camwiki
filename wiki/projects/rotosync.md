---
title: "Rotosync"
type: project
tags: [project, standup, meetings, firebase, daily, transcription, ai]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
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

| Type | Description |
|------|-------------|
| Daily Standup | Team rotation, speaker turns, wrap-up sections |
| 1:1 | Two-person meeting with focused summary format |
| Project Meeting | Project-specific discussions |
| Ad-hoc | Custom attendee selection, quick meetings |

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

Rotosync appears as a conferencing option on calendar events via a Google Workspace add-on (`meet-addon/`). One-click launch from calendar events.

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

## Open Questions

_No entries yet._

## Sources

- [[wiki/sources/rotosync-repo]] — meeting types, AI pipeline, Monday.com integration, Firebase architecture
