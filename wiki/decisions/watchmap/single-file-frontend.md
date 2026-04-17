---
title: "Decision: Single-file frontend (no build step)"
type: decision
tags: [decision, adr, frontend, no-build, leaflet]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Single-file frontend (no build step)

**Date:** 2026-04-17 (reconstructed from initial commit 5b2c94f)
**Status:** `accepted`

## Context

Watchmap's UI is a map dashboard — Leaflet.js map, Socket.IO client, session sidebar, system stats. The question was whether to use a framework with a build step (React/Vue) or keep it as a single HTML file.

## Options Considered

1. **React/Vue + bundler** — component model, npm dependencies, hot module replacement. Standard for larger apps but adds a build step, `node_modules`, and deployment complexity.
2. **Single `index.html`** — inline CSS and JS, Leaflet and Socket.IO loaded from CDN. No build step, no artifacts, volume-mounted for instant reload.

## Decision

Single `frontend/index.html` with inline styles and scripts. Served by nginx:alpine via Docker volume mount.

Key reasons:
- **Zero build artifacts:** Nginx serves the HTML file directly. Changes are live on container restart without rebuilding — documented explicitly in CLAUDE.md.
- **Zero frontend dependencies:** Leaflet.js and Socket.IO client loaded from CDN; no `npm install` needed.
- **Readability:** Entire UI is one file — maintainable for a single-developer dashboard project.
- **Fast iteration:** Edit file → restart nginx container → see change. No build pipeline to wait for.

## Consequences

- Scales poorly beyond ~1000 lines — no component model, no scoping, no TypeScript. Acceptable for a dashboard of this complexity.
- CDN dependency: if Leaflet or Socket.IO CDN is unavailable, the dashboard breaks. Mitigated by the fact that the home server is internet-connected.

## Related Pages

- [[wiki/projects/watchmap]]

## Sources

- `raw/repos/watchmap` — frontend/index.html, docker-compose.yml, CLAUDE.md
