---
title: "Decision: Multi-meeting-type architecture with title-pattern detection"
type: decision
tags: [decision, adr, architecture, meeting-types]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Multi-meeting-type architecture with title-pattern detection

**Date:** 2026-04-17 (reconstructed from repo)
**Status:** `accepted`

## Context

Rotosync started as a standup tool. As usage grew, it needed to support 1:1 meetings and project meetings with different UX, data scoping, and AI extraction behaviour. The question was how to detect and route meeting types without requiring users to configure anything.

## Options Considered

1. **Single meeting type** — all meetings behave identically. Simple but produces wrong UX for 1:1s (no rotation panel needed) and wrong data scoping (project carryover items bleeding into standups).
2. **User-selected type** — user picks type when adding conferencing. Adds friction; prone to misconfiguration.
3. **Title-pattern detection** — infer type from calendar event title: `[ProjectName]` tag → project, "standup" keyword → standup, else → 1:1/open. Store result in Firestore.

## Decision

Title-pattern detection via `meetingTypeDetector.ts`, with type stored in Firestore on room creation. Detection order: project tag → standup keyword → 1:1 default.

**Default-to-1:1 design:** Unknown/unrecognised meeting titles fall through to 1:1 mode — the simplest variant. This preserves backward compatibility (existing standups have "standup" in the title) and ensures new meeting types degrade gracefully.

**Carryover visibility matrix:**

| Item created in | Shows in standup | Shows in project | Shows in 1:1 |
|-----------------|-----------------|-----------------|-------------|
| Standup | ✓ | ✗ | ✓ (if attendee matches) |
| Project | ✗ | ✓ | ✓ (if attendee matches) |
| 1:1 | ✗ | ✗ | ✓ (only same pair) |

1:1 items store `oneOnOneAttendees` (sorted email pair) to prevent cross-contamination between different 1:1 pairs.

## Consequences

- Carryover visibility logic is complex — enforced in both webapp (`useCarryover.ts`) and Cloud Functions (`summarizeStandup.ts`); must stay in sync.
- If a meeting title changes (e.g., "standup" removed), type detection changes and existing carryover items may not appear.
- Backfill migration scripts needed when new fields (e.g., `oneOnOneAttendees`) are added to existing documents.

## Related Pages

- [[wiki/projects/rotosync]]
- [[wiki/decisions/rotosync/email-scoped-carryover]]

## Sources

- `raw/repos/rotosync` — functions/src/meetingTypeDetector.ts, PLAN-multi-meeting-types.md, CLAUDE.md
