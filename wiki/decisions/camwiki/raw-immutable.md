---
title: "Decision: raw/ directory is strictly immutable"
type: decision
tags: [decision, adr, raw, immutable, source-of-truth]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Decision: raw/ directory is strictly immutable

**Date:** 2026-04-17 (reconstructed from CLAUDE.md)
**Status:** `accepted`

## Context

The wiki has two layers: `raw/` (source documents) and `wiki/` (synthesized pages). Allowing Claude to modify raw sources would blur the line between original artifacts and LLM synthesis, breaking auditability.

## Options Considered

1. **Mutable raw layer** — Claude normalizes, annotates, or cleans raw documents. Synthesis and originals become entangled; re-ingestion from a clean source is impossible.
2. **Immutable raw layer** — Claude reads raw documents but never writes to them. The synthesized `wiki/` layer is entirely Claude-owned; `raw/` is entirely human-owned.

## Decision

`raw/` is strictly immutable. `CLAUDE.md` Section 2 hard rule: *"Never create, modify, or delete anything under `raw/`."*

Key reasons:
- **Auditability:** If a wiki page is wrong, the original raw source is still there, unchanged, to audit against.
- **Replay:** If synthesis rules change (new templates, new workflows), raw documents can be re-ingested cleanly without any data loss or contamination.
- **Source of truth:** Raw documents are the authoritative original. Wiki pages are derived artifacts — they inherit their authority from `raw/`.
- **Git repos symlinked:** Live git repos are symlinked under `raw/repos/` and accessed via `git log`, `git diff`, and direct file reads. The symlink pattern reinforces read-only access.

## Consequences

- Raw documents do not benefit from LLM cleanup or normalisation — they stay in their original form.
- When raw sources are updated (new commits, new docs), the wiki must be manually re-ingested — changes don't propagate automatically.

## Related Pages

- [[wiki/projects/camwiki]]
- [[wiki/decisions/camwiki/pre-synthesis-over-rag]]

## Sources

- `CLAUDE.md` — Section 2, Section 3
