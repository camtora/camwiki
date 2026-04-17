---
title: "Decision: Expo Push API instead of Firebase Cloud Messaging"
type: decision
tags: [decision, adr, push-notifications, expo, firebase]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Decision: Expo Push API instead of Firebase Cloud Messaging

**Date:** 2026-04-16 (reconstructed from repo)
**Status:** `accepted`

## Context

The SBCA app sends push notifications for two events: dues expiry warnings (30 days before) and significant water level changes (±30 cm/week). Expo and Firebase/FCM are the two mainstream options for React Native push notifications.

## Options Considered

1. **Firebase Cloud Messaging (FCM)** — industry standard. Requires a Firebase project, Google Services configuration in the Android build, and Apple Push Notification service (APNs) certificates for iOS. Two separate services to configure (Google for Android, Apple for iOS), unified via FCM.
2. **AWS SNS** — paid; overkill for a cottage association with a small member base.
3. **Expo Push API** — Expo's free hosted service that abstracts FCM (Android) and APNs (iOS) behind a single endpoint. Tokens look like `ExponentPushToken[xxx...]`. One HTTP call to `https://exp.host/--/api/v2/push/send` handles both platforms.

## Decision

Expo Push API. The free tier covers SBCA's usage entirely. More importantly: no Firebase project setup, no `google-services.json` in the Android build, no separate APNs certificate management. The backend simply POSTs an array of push objects to Expo's endpoint.

Device tokens registered via `POST /push/register` and stored per member. Cron jobs (`checkDuesExpiry`, `syncWaterLevels`) use a shared `sendPushToAll()` helper.

Notifications include `data: { screen: 'membership/renew' }` for deep linking — the app routes to the relevant screen on tap.

## Consequences

- Single Expo account dependency for push delivery; if Expo's push service has downtime, notifications are delayed.
- Cannot use FCM-specific features (e.g., topic subscriptions, FCM data-only messages for background processing).
- Acceptable trade-off for a small-membership community app where push is advisory, not mission-critical.

## Related Pages

- [[wiki/projects/sbca]]

## Sources

- `raw/repos/sbca` — app/services/notifications.ts, backend/src/routes/push.ts, backend/src/jobs/checkDuesExpiry.ts, docker-compose.yml (EXPO_PUSH_URL)
