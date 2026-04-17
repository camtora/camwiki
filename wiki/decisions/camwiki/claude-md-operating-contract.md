---
title: "Decision: CLAUDE.md as self-activating operating contract"
type: decision
tags: [decision, adr, claude-md, operating-contract]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Decision: CLAUDE.md as self-activating operating contract

**Date:** 2026-04-17 (reconstructed from CLAUDE.md)
**Status:** `accepted`

## Context

An LLM-maintained wiki needs consistent behaviour across sessions: same naming conventions, same page templates, same workflows. The question is where to encode the rules and how to ensure Claude always reads them.

## Options Considered

1. **No contract file** — Claude learns conventions through conversation each session; human re-explains rules when they're violated. Fragile; inconsistency accumulates over sessions.
2. **Rules scattered across wiki pages** — conventions documented in various places. Hard to discover; no guarantee Claude reads them.
3. **`CLAUDE.md` at repo root** — Claude Code automatically reads `CLAUDE.md` at session start. All rules in one file; self-activating; human-readable.

## Decision

`CLAUDE.md` at repo root as the single operating contract. It is the first thing read every session (Section 10 Session Startup Checklist: *"Do not skip step 1 even if you believe you remember the conventions"*).

The file encodes: directory layout, hard rules, naming conventions, frontmatter schema, 7 page templates, 3 workflows, index entry format, log entry format, and the session startup checklist. If `CLAUDE.md` and a user instruction conflict, Claude asks for clarification before proceeding.

When rules change (e.g., the `wiki/decisions/` nesting exception), `CLAUDE.md` is updated as the authoritative source.

## Consequences

- All rule changes must be made in `CLAUDE.md` — not in conversation, not in wiki pages.
- The operating contract is version-controlled via git — rule evolution is auditable.
- `CLAUDE.md` must be kept under ~500 lines or it becomes unwieldy to read in full at session start.

## Related Pages

- [[wiki/projects/camwiki]]
- [[wiki/decisions/camwiki/append-only-log]]

## Sources

- `CLAUDE.md` — Sections 1, 10
