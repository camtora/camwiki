---
title: "Decision: Firebase as unified backend platform"
type: decision
tags: [decision, adr, firebase, firestore, cloud-functions]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Firebase as unified backend platform

**Date:** 2026-04-17 (reconstructed from repo)
**Status:** `accepted`

## Context

Rotosync requires real-time multi-user collaboration (live transcripts, active speaker tracking, team rotation), Google Calendar integration, and a lightweight deployment model. The question was whether to use a conventional SQL backend or Firebase's unified cloud platform.

## Options Considered

1. **PostgreSQL + Supabase** — standard relational backend. Easier ad-hoc queries and inspection; more familiar to SQL users. But requires separate real-time layer, separate auth, and separate hosting — three systems.
2. **Firebase (Firestore + Cloud Functions + Auth + Hosting)** — unified platform; one deployment command (`firebase deploy`); Firestore's real-time listeners align with live standup requirements.

## Decision

Firebase for all persistence, compute, auth, and hosting. Key reasons:

- **Real-time native:** Firestore snapshot listeners push updates to all connected clients instantly — active speaker changes, carryover item updates, and session state all flow via listeners with no polling.
- **Multi-user collaboration:** Firebase Auth integrates directly with Firestore security rules; no separate user management layer.
- **Unified secrets and IAM:** Authentication, database, functions, and hosting share one GCP project, one billing account, one `firebase deploy` command.
- **Apps Script embedding:** Google Calendar Add-on callbacks directly trigger Cloud Functions; no separate API server to manage.
- Explicitly documented in `docs/tech-stack-comparison.md`: Firebase wins for "real-time, multi-user, video-enabled" products.

## Consequences

- Ad-hoc queries and data inspection are harder than SQL — no `SELECT *` equivalent; data lives in Firestore documents.
- All backend logic runs in Cloud Functions (TypeScript) with cold-start latency on first invocation.
- Real-time listeners mean clients receive updates without polling — simpler client code, but Firestore billing scales with reads.

## Related Pages

- [[wiki/projects/rotosync]]

## Sources

- `raw/repos/rotosync` — firebase.json, docs/tech-stack-comparison.md, CLAUDE.md
