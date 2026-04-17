---
title: "Decision: iOS-only launch with SwiftUI + MapKit"
type: decision
tags: [decision, adr, ios, swiftui, mapkit, platform]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: iOS-only launch with SwiftUI + MapKit

**Date:** 2026-04-16 (reconstructed from docs/IOS.md and repo structure)
**Status:** `accepted`

## Context

Who's Up is a location-based social app. The platform choice (iOS, Android, cross-platform, web) shapes the entire development approach. The initial launch targets 2–3 geographic areas (Muskoka, Santa Teresa, Toronto) with a small initial user base.

## Options Considered

1. **React Native / Flutter (cross-platform)** — single codebase for iOS + Android. More reach, but Stripe Terminal library maturity on React Native was a concern at the time; also loses native platform integration quality.
2. **Android-first** — larger global market share, but demographics for Muskoka/Santa Teresa skew toward iPhone users.
3. **Web app (PWA)** — no install friction, but Core Location and background location access are severely limited in mobile browsers; can't match native UX.
4. **iOS-only SwiftUI** — native iOS 17+. Full access to Core Location, Sign in with Apple (required by App Store for social apps), MapKit, and Keychain. Single platform allows focused MVP.

## Decision

iOS-only with SwiftUI (iOS 17+). Sign in with Apple is required by App Store guidelines for apps offering social login — mandating Apple as an auth option simplified the auth strategy (Apple + Google only; no passwords). MapKit was chosen over Google Maps and Mapbox — no API key management, native iOS ecosystem integration, and Siri Remote support (relevant for the [[wiki/projects/watchmap]] tvOS app in a related project).

SwiftUI MVVM: Views are pure UI; `EnvironmentObject` services (`AuthManager`, `LocationManager`, `PresenceManager`, `APIClient`) are injected at app root. Tab-based navigation with 5 tabs: Join (map) → Host → Activities → Connections → Profile.

Location permission: "When In Use" only — not "Always" — respects privacy and reduces permission friction on install.

## Consequences

- Android users in launch regions are excluded. Accepted trade-off for MVP focus.
- App Store requires Apple Developer Program ($99/year) for physical device testing and distribution.
- Sign in with Apple token signature verification noted as tech debt (commit comment: "production should verify with Apple") — currently basic JWT decode.
- SwiftUI MVVM keeps UI/logic separation clean; `EnvironmentObject` pattern allows services to be swapped for testing stubs.

## Related Pages

- [[wiki/projects/whosup]]

## Sources

- `raw/repos/whosup/docs/IOS.md` — app architecture, services, navigation, tab structure
- `raw/repos/whosup/docs/ARCHITECTURE.md` — system overview, JWT storage in Keychain
