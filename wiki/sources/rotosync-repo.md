---
title: "Source: rotosync repo"
type: source
tags: [source, standup, firebase, daily, transcription, ai, monday]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: rotosync repo

**Raw file:** `raw/repos/rotosync`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

Rotosync is a standup facilitation webapp: React/Vite frontend + Firebase Cloud Functions + Firestore + Daily.co WebRTC. Supports daily standups, 1:1s, project meetings, and ad-hoc meetings. Live transcription with per-speaker attribution. AI summaries switched from Claude to Gemini/Vertex AI. Monday.com integration for data tickets and CAB board. Google Meet add-on for calendar integration. Several standup transcripts committed to the repo from Jan 2026.

## Key Facts / Data Points

- AI switched from Claude to Gemini (Vertex AI): commit `95a6658` (2026-04-09)
- Daily.co WebRTC for audio/video; audio-only by default
- Monday.com API used in two wrap-up sections: Data Tickets (unassigned) and CAB Board (charity launches)
- Gmail API with domain-wide delegation for post-meeting email summaries
- Carryover system: blockers + action items persist across standups until resolved
- Hosted on Firebase — no home server dependency (`rotosync-7b249.web.app`)
- Google Meet addon in `meet-addon/` — adds Rotosync as a conferencing option in Google Calendar

## Pages Updated

- [[wiki/projects/rotosync]] — created
