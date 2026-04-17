---
title: "Decision: Native tvOS SwiftUI app instead of WebKit wrapper"
type: decision
tags: [decision, adr, tvos, swiftui, mapkit, webkit]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Decision: Native tvOS SwiftUI app instead of WebKit wrapper

**Date:** 2026-04-17 (reconstructed from commit fab62ea)
**Status:** `accepted`

## Context

Watchmap has a web dashboard that works on desktop and mobile. The goal was to display it on an Apple TV for an always-on TV display. The question was whether to wrap the existing web UI in a WebView or build a native tvOS app.

## Options Considered

1. **WKWebView wrapper** — display the existing `index.html` in a tvOS WebView app. Zero duplication; reuses all existing frontend code. But: `WKWebView` framework is not available on tvOS — Apple restricts it to internal use only.
2. **Native SwiftUI + MapKit** — rebuild the dashboard as a native tvOS app using MapKit for the map and polling the backend REST API.

## Decision

Native SwiftUI app (`WatchMapTV`) using MapKit. Polls REST API endpoints (`/api/locations`, `/api/streams`, `/api/metrics`) every 5 seconds.

Key reasons beyond the WebKit constraint:
- **Better TV UX:** Full-screen without browser chrome; Siri Remote supported natively; `isIdleTimerDisabled = true` keeps screen on.
- **Z-ordering control:** `MapReader` overlay ensures active stream markers always render on top — not achievable in the web Leaflet map without significant rework.
- **Simpler integration:** REST API polling every 5 seconds is simpler than implementing a Socket.IO Swift client (which would require a third-party dependency).

The 5-second poll interval (vs. 2-second Socket.IO push in the web app) is a deliberate trade-off: slightly less real-time, but acceptable for a TV display where viewers aren't watching millisecond changes.

## Consequences

- UI logic is duplicated between `frontend/index.html` (web) and `WatchMapTV/` (tvOS). Changes to the dashboard design require updates in both places.
- `NSAllowsArbitraryLoads = true` in `Info.plist` allows HTTP access to the home server backend (which doesn't have HTTPS on the local network).

## Related Pages

- [[wiki/projects/watchmap]]
- [[wiki/concepts/websocket-realtime-events]]

## Sources

- `raw/repos/watchmap` — WatchMapTV/ContentView.swift, docs/HISTORICAL_MARKERS_AND_TVOS.md, commit fab62ea
