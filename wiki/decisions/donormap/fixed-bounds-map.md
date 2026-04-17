---
title: "Decision: Fixed-bounds map with no zoom or pan"
type: decision
tags: [decision, adr, leaflet, map, broadcast]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Fixed-bounds map with no zoom or pan

**Date:** 2026-04-17 (reconstructed from repo)
**Status:** `accepted`

## Context

DonorMap is designed for **broadcast display** — office TV walls and staff dashboards showing live donor acquisition activity across North America. The question was whether to give users interactive map controls.

## Options Considered

1. **Full interactive Leaflet controls** — zoom, pan, click to inspect. Standard map UX. But: users might miss live updates while navigating; hard to synchronise view across two TVs; TV remote D-pad can't do precise zoom.
2. **Fixed bounds, country toggle** — locked viewport for US and Canada; country toggle switches between fixed maps. Consistent view across all display devices.

## Decision

Fixed-bounds Leaflet map: `setView()` locks viewport, no zoom controls rendered. Country toggle switches between pre-configured US and CA bounds. Maps automatically fit to active donor markers daily.

- Optimised for **broadcast, not exploration** — the value is seeing dots move in real-time, not navigating.
- TV remote support: D-pad can operate the country toggle; no precise zoom/pan interaction required.
- Consistent view: all office TVs and individual dashboards show the same bounded view simultaneously.

## Consequences

- Cannot drill down to city level. Accepted — FSA-level choropleth (postal code area) provides sufficient geographic granularity for the use case.
- Fire TV app (`firetv-app/`) uses fixed zoom (0.75x) for wall broadcast optimisation.

## Related Pages

- [[wiki/projects/donormap]]
- [[wiki/decisions/donormap/firetv-pairing-code-auth]]

## Sources

- `raw/repos/donormap` — webapp/src/components/DonorMap.tsx, firetv-app/README.md, commit e2b2fe9
