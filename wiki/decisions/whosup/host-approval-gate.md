---
title: "Decision: Host approval gate for activity joins"
type: decision
tags: [decision, adr, safety, ux, social]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Host approval gate for activity joins (PENDING → APPROVED/DENIED)

**Date:** 2026-04-16 (reconstructed; commit 93047ae "Implement activity join flow with host approval")
**Status:** `accepted`

## Context

When a user discovers an activity on the map and wants to join, the system needed a policy for how that join happens. This is a social safety decision as much as a technical one.

## Options Considered

1. **Auto-join** — any user can join any activity instantly. Maximally frictionless. Host has no control over who shows up.
2. **First-come-first-served** — first N requesters auto-approved up to `maxParticipants`. Simple but host still has no control.
3. **Host selects from a discovery list** — host browses a list of potential joiners and invites them. Inverts the interaction; requires host to be active.
4. **Requester sends join request; host approves or denies** — PENDING state while host reviews. Requester sees profile of host; host sees profile, photos, and optional message from requester before deciding.

## Decision

Explicit host approval gate. State machine: `PENDING → APPROVED` or `PENDING → DENIED`.

On approval: requester is added to `RoomMember`, gains access to the group chat, and the `latExact`/`lngExact` coordinates are disclosed (they can now navigate to the exact meeting point). On denial: requester sees the denial; can't re-request the same presence.

The host receives a real-time Socket.io `new-request` event and sees the requester's full profile (display name, photos, bio, age range, mutual connections) before deciding. Requester can include an optional message (max 500 chars) with the request.

Rationale: "Discovery" (finding someone on the map) and "trust" (showing up at their location) are different states. The approval gate is the explicit transition between them. It also gives the host safety control — they can review a profile and decline without explanation.

## Consequences

- Activities with an absent host become orphaned (no one to approve). Mitigated by `expiresAt` — presence auto-expires from the map.
- Blocked users get a 404 on join requests — doesn't reveal the host's existence to a blocked user.
- Approval adds to `RoomMember` — group chat access and exact location are bundled. There is no partial approval.
- Multiple non-overlapping activities per user are allowed; overlap is checked by `startAt`/`expiresAt` window.

## Related Pages

- [[wiki/projects/whosup]]
- [[wiki/decisions/whosup/location-privacy-three-layers]]

## Sources

- `raw/repos/whosup/docs/ARCHITECTURE.md` — "Join Request Flow" sequence diagram
- `raw/repos/whosup/docs/DATABASE.md` — ActivityRequest model, status enum
- `raw/repos/whosup` — commit 93047ae, backend/src/routes/activity-requests.ts
