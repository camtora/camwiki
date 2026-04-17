---
title: "Decision: Append-only log.md as immutable audit trail"
type: decision
tags: [decision, adr, log, audit, append-only]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Decision: Append-only log.md as immutable audit trail

**Date:** 2026-04-17 (reconstructed from CLAUDE.md)
**Status:** `accepted`

## Context

The wiki evolves over many sessions. Without a record, it's impossible to know what changed, when, or why — making it hard to audit page content or understand the wiki's history.

## Options Considered

1. **No log** — silent operations; no audit trail. Can't explain why a page contains certain information.
2. **Mutable changelog** — update entries as understanding improves. Loses original intent; entries become revisionist.
3. **Append-only log** — one entry per operation (ingest, query, lint, edit); past entries never modified or deleted.

## Decision

`log.md` at repo root: append-only, one entry per operation, structured format. `CLAUDE.md` Section 11 General Principles: *"Preserve the log's integrity. Never edit past log entries."*

Each entry records: date, operation type, source (for ingests), pages created, pages updated, and notes on anything notable or unresolved. Grep patterns for common queries are documented in Section 9:
- All ingests: `grep "^\#\# \[" log.md | grep "ingest"`
- Last 5 operations: `grep "^\#\# \[" log.md | tail -5`

Combined with version control (`git log`), `log.md` provides two independent audit layers: one semantic (what the operation was), one structural (what files changed).

## Consequences

- `log.md` grows indefinitely — acceptable since it's append-only text.
- Past errors or misunderstandings are preserved in the log rather than silently corrected — which is a feature, not a bug, for an audit trail.
- The log is only useful if entries are written at every operation — discipline required.

## Related Pages

- [[wiki/projects/camwiki]]
- [[wiki/decisions/camwiki/claude-md-operating-contract]]

## Sources

- `CLAUDE.md` — Sections 9, 11
