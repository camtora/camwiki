---
title: "Decision: Daily.co for embedded WebRTC and live transcription"
type: decision
tags: [decision, adr, daily-co, webrtc, transcription, video]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Daily.co for embedded WebRTC and live transcription

**Date:** 2026-04-17 (reconstructed from repo)
**Status:** `accepted`

## Context

Rotosync needs video/audio conferencing with real-time transcription and active speaker detection — the transcript feeds the AI extraction pipeline. The question was whether to use an existing meeting platform (Meet, Zoom) or an embedded WebRTC SDK.

## Options Considered

1. **Google Meet / Zoom API** — rely on existing platforms users already have. But: no real-time transcript access; no active speaker events; meeting happens outside the app, breaking the integrated flow.
2. **Custom WebRTC** — full control but substantial infrastructure burden (TURN/STUN servers, signalling, transcription service).
3. **Daily.co** — embeddable WebRTC SDK with native live transcription and speaker attribution, API-driven room management.

## Decision

Daily.co for all conferencing. Key reasons:

- **Live transcription built-in:** Daily.co provides real-time transcript events with speaker attribution; `useTranscript.ts` consumes these directly. No separate speech-to-text service.
- **Active speaker detection:** Daily.co emits participant events (mute/unmute, active speaker) consumed by `useActiveSpeaker.ts` for automatic rotation advancement.
- **API-driven room management:** Cloud Functions create rooms and issue tokens per-request; users never see raw meeting URLs.
- **Audio-only by default:** Standup meetings use `start_video_off` for focus and reduced bandwidth. Video is enabled for 1:1 meetings via per-type configuration in `PLAN-multi-meeting-types.md`.
- **Screen sharing included** — `ScreenShareView.tsx` with no additional infrastructure.

## Consequences

- Vendor lock-in: Rotosync's transcript and active speaker logic is tightly coupled to Daily.co's event model.
- Daily.co billing scales with meeting minutes.
- Room creation must happen server-side (Cloud Functions) to keep API keys off the client.

## Related Pages

- [[wiki/projects/rotosync]]
- [[wiki/decisions/rotosync/firebase-unified-backend]]

## Sources

- `raw/repos/rotosync` — webapp/package.json, functions/src/createConference.ts, CLAUDE.md, PLAN-multi-meeting-types.md
