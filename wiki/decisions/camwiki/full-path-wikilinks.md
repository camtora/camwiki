---
title: "Decision: Full-path wiki-links instead of relative or bare links"
type: decision
tags: [decision, adr, wikilinks, obsidian, cross-referencing]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Decision: Full-path wiki-links instead of relative or bare links

**Date:** 2026-04-17 (reconstructed from CLAUDE.md)
**Status:** `accepted`

## Context

Wiki-internal links can be written three ways in Obsidian: bare filename (`[[page-name]]`), relative path (`[[../projects/page-name]]`), or full path from repo root (`[[wiki/projects/page-name]]`). The choice affects collision safety and refactoring resilience.

## Options Considered

1. **Bare filenames** (`[[page-name]]`) — short and readable. But collisions are inevitable as the wiki scales to org-level (two teams might both have a `deployment.md`).
2. **Relative paths** (`[[../projects/page-name]]`) — avoids collisions but breaks when pages are moved to different directories.
3. **Full paths from repo root** (`[[wiki/projects/page-name]]`) — unambiguous, collision-safe, refactoring-safe.

## Decision

Full-path wiki-links only: `[[wiki/category/page-slug]]`. Display aliases when helpful: `[[wiki/projects/home-media-server|Media Server]]`.

Key reasons:
- **No collisions:** The wiki is designed to scale to org-level covering multiple teams and codebases. Full paths prevent name collisions across categories and teams.
- **Refactoring-safe:** Full paths are absolute; they don't break when intermediate directories are added or renamed.
- **Machine-readable:** Full-path links are unambiguous for tooling — grep, lint checks, and index validation all work reliably.

When the `wiki/decisions/` directory was restructured into per-project subdirectories, all existing links were updated via `sed` — the full-path convention made this mechanical rather than ambiguous.

## Consequences

- Links are verbose compared to bare filenames — `[[wiki/decisions/infra/decision-dns-failover-approach]]` vs `[[decision-dns-failover-approach]]`.
- Never use relative paths — they break across directory moves, which are a normal part of wiki restructuring.

## Related Pages

- [[wiki/projects/camwiki]]

## Sources

- `CLAUDE.md` — Section 4 (Naming Conventions)
