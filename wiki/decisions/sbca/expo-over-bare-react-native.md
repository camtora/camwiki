---
title: "Decision: Expo SDK over bare React Native"
type: decision
tags: [decision, adr, expo, react-native, mobile]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Expo SDK over bare React Native

**Date:** 2026-04-16 (reconstructed from repo)
**Status:** `accepted`

## Context

The SBCA app is a React Native project developed on Ubuntu with iOS builds compiled on a Mac via SSHFS mount. The choice was between Expo (managed/prebuild workflow) and bare React Native.

## Options Considered

1. **Bare React Native** — full native control, no Expo layer. Requires manual iOS/Android project setup. Build tools must run natively on Mac or in CI.
2. **Expo SDK + Expo Router** — managed SDK with file-based routing. `expo prebuild` generates native artifacts on demand; Metro bundler runs on Ubuntu; Xcode build happens on Mac.
3. **Flutter / native Swift** — rejected early; JS monorepo with backend sharing is preferred.

## Decision

Expo SDK 55 with Expo Router (file-based routing). Key drivers:

- **Ubuntu dev loop**: Metro bundler runs on Ubuntu (`metro.sba.camerontora.ca:8081`); only native compilation requires Mac. Bare RN would require constant context switching for even minor JS changes.
- **No Firebase dependency**: Expo Push API handles push notifications without FCM — no Google Play Services dependency, no Firebase project to manage.
- **Offline-first primitives**: `expo-secure-store`, `@react-native-async-storage/async-storage`, and `expo-notifications` are managed plugins — no manual native linking.
- **`expo prebuild` as escape hatch**: When native code is needed (Stripe CardField, CocoaPods), `expo prebuild --platform ios` generates the full Xcode project without abandoning the Expo workflow.

iOS build chain: `expo prebuild` on Ubuntu → `pod install` on Mac (not over SSHFS mount — write errors) → open `SBCA.xcworkspace` in Xcode (workspace, not `.xcodeproj`).

## Consequences

- EAS Build (Expo's cloud build service) not used — free `expo prebuild` with local Xcode gives full control and faster debugging.
- Expo SDK version pins all native dependencies; upgrading requires coordinated SDK bump.
- Expo Router provides the same file-based routing pattern as Next.js — consistent mental model across the monorepo.

## Related Pages

- [[wiki/projects/sbca]]

## Sources

- `raw/repos/sbca` — app/package.json, app/app.json, DEVELOPMENT.md (iOS build workflow)
