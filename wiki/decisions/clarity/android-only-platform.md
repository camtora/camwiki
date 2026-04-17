---
title: "Decision: Android-only platform for door-to-door fundraising"
type: decision
tags: [decision, adr, android, platform, mobile]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Android-only platform for door-to-door fundraising

**Date:** 2026-04-16 (reconstructed from repo structure and manifest)
**Status:** `accepted`

## Context

Clarity is a tablet app for GlobalFaces Direct fundraisers who knock on doors. It needed to support NFC tap-to-pay, Bluetooth peripherals, and GPS. The platform choice determines hardware costs and Stripe feature availability.

## Options Considered

1. **iOS (iPad)** — premium build quality, but iPads cost significantly more than Android tablets for bulk procurement; Stripe's Tap to Pay on iPhone launched later than Android's equivalent.
2. **Cross-platform (React Native / Flutter)** — single codebase for both. Stripe Terminal SDK maturity on React Native was a concern; also adds abstraction layer over NFC and Bluetooth APIs that need reliable, field-tested behavior.
3. **Android-only** — enterprise Android tablets are substantially cheaper for bulk fundraiser deployments; Stripe Tap to Pay on Android was available and tested; NFC, Bluetooth, and Location APIs are well-established.

## Decision

Android-only, targeting minSdk 26 (Android 8.0+), targetSdk 34 (Android 14). Reasons:

- **Stripe Tap to Pay on Android**: NFC card reading using the device's built-in NFC chip was available and production-ready. No external hardware reader needed — each fundraiser tablet is self-contained.
- **Tablet cost**: Android tablets are dominant in enterprise/institutional deployments; lower per-unit cost for bulk fundraiser provisioning.
- **Permission model**: Android 8+ (minSdk 26) provides NFC, Bluetooth, and Location permissions without legacy APIs; Android 12+ Bluetooth permission split handled explicitly in `MainActivity.kt`.
- **Stripe ecosystem**: Stripe Terminal SDK on Android was mature and actively supported.

## Consequences

- iOS users cannot use the fundraiser app. Acceptable — deployment is to company-issued devices, not BYOD.
- Requires `android.hardware.nfc` as a required feature in the manifest; devices without NFC cannot install the app.
- targetSdk 34 required by Google Play policy (late 2024); handled.
- BLUETOOTH_SCAN and BLUETOOTH_CONNECT permissions handled separately for SDK ≥ 31 vs older devices.

## Related Pages

- [[wiki/projects/clarity]]
- [[wiki/decisions/clarity/stripe-tap-to-pay]]

## Sources

- `raw/repos/clarity` — app/build.gradle.kts (minSdk, targetSdk, Stripe Terminal libs), AndroidManifest.xml (NFC required feature, permissions), MainActivity.kt (permission handling)
