---
title: "Expo / React Native Build Pipeline"
type: concept
tags: [concept, expo, react-native, ios, mobile, build]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Expo / React Native Build Pipeline

Expo is a framework and toolchain for React Native that abstracts native iOS/Android project configuration. The managed workflow handles native config via `app.json`; `expo prebuild` generates the native `ios/` and `android/` directories when needed. Metro is the JavaScript bundler that serves the app during development.

## How It Is Used Here

[[wiki/projects/sbca\|SBCA]] uses Expo SDK 55 with a split dev environment — code lives on the Ubuntu server, native tooling runs on Mac. This is the canonical pattern for this stack.

## Key Properties / Mechanics

### The Ubuntu + Mac Split

| Task | Machine | Why |
|------|---------|-----|
| Source code editing | Ubuntu server | Claude Code, all `npm install`, Metro bundler |
| iOS Simulator / Xcode | MacBook | Xcode requires macOS; no Linux support |
| CocoaPods (`pod install`) | MacBook | CocoaPods resolves iOS native dependencies |
| Android Studio | MacBook | Optional; Android emulator also works here |

Files are shared between machines via SSHFS (`~/mnt/HOMENAS`) — the Ubuntu source tree is mounted on the Mac. **Never run `npm install` on Mac** — `node_modules` must stay on Ubuntu where Metro runs.

### Metro Bundler

Metro is the JS bundler that React Native apps connect to during development. It runs on Ubuntu (port 8081) and is exposed via a subdomain:

```
metro.sba.camerontora.ca → Ubuntu:8081
```

The iOS Simulator on Mac connects to this URL. Using a subdomain (rather than a raw IP) allows the wildcard `*.sba` SSL cert to cover it and avoids iOS ATS (App Transport Security) issues with plain HTTP.

### `expo prebuild`

Generates the `ios/` and `android/` native directories from `app.json`. Must be run when:
- Adding a new native module (e.g., a library with Objective-C/Swift code)
- Upgrading Expo SDK version
- Changing native config in `app.json`

After prebuild, run `pod install` on Mac (not over SSHFS — native file operations are slow over the network mount).

### Opening in Xcode

Always open `ios/<AppName>.xcworkspace`, **not** `ios/<AppName>.xcodeproj`. The `.xcworkspace` includes CocoaPods dependencies; the `.xcodeproj` alone will fail to build.

### Expo Router

SBCA uses Expo Router (file-based routing), analogous to Next.js App Router. Pages live in `app/app/` — each file is a route. Navigation is handled automatically.

### Push Notifications

SBCA uses the **Expo Push API** rather than Firebase Cloud Messaging (FCM) directly. A single endpoint (`https://exp.host/--/api/v2/push/send`) handles delivery to both iOS and Android. No Firebase project, no `google-services.json`, no native SDK configuration beyond Expo's built-in support. See [[wiki/decisions/sbca/expo-push-not-firebase]].

### EAS Build (not used)

Expo Application Services (EAS) Build is Expo's cloud build service. SBCA uses local builds (Mac Xcode) rather than EAS — avoids EAS subscription cost and keeps the build pipeline local.

## Relationships to Other Concepts

- [[wiki/concepts/jwt-authentication]] — SBCA stores JWTs in Expo SecureStore
- [[wiki/concepts/stripe-payment-flows]] — Stripe React Native SDK integrated into the Expo app
- [[wiki/concepts/webhook-patterns]] — push notification delivery goes through Expo's servers

## External References

- Expo docs: https://docs.expo.dev
- Expo Router: https://expo.github.io/router/docs
