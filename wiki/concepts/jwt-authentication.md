---
title: "JWT Authentication"
type: concept
tags: [concept, auth, jwt, mobile, security]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# JWT Authentication

JSON Web Tokens (JWT) are self-contained, signed tokens used to authenticate API requests. The server issues a token on login; the client includes it in subsequent requests. The server verifies the signature without a database lookup — stateless authentication.

## How It Is Used Here

Two projects implement JWT auth independently (neither uses [[wiki/concepts/oauth2-proxy-pattern]] — both are public-facing apps with their own user bases):

| Project | Token lifetime | Storage | Refresh strategy |
|---------|---------------|---------|-----------------|
| [[wiki/projects/sbca\|SBCA]] | Not documented | Expo SecureStore | Access + refresh token pair |
| [[wiki/projects/whosup\|Whosup]] | 7 days | iOS Keychain | 7-day lifetime; re-login on expiry |

## Key Properties / Mechanics

### Token Structure

A JWT has three base64url-encoded parts: `header.payload.signature`.

- **Header:** algorithm (typically `HS256` or `RS256`) and token type
- **Payload:** claims — `sub` (user ID), `exp` (expiry), `iat` (issued at), plus any custom fields
- **Signature:** HMAC or RSA signature over header + payload using the server's secret

The server verifies by re-computing the signature. If it matches and `exp` is in the future, the token is valid — no DB lookup required.

### Access + Refresh Token Pattern (SBCA)

Two tokens are issued on login:

- **Access token** — short-lived (minutes to hours); sent with every API request in `Authorization: Bearer <token>`
- **Refresh token** — long-lived (days to weeks); stored securely; used only to obtain a new access token when the current one expires

Axios interceptors in SBCA auto-attach the access token and handle 401 responses by transparently refreshing and retrying.

### Single Long-lived Token (Whosup)

Whosup uses a 7-day JWT with no refresh mechanism. Simpler implementation; the trade-off is that a stolen token is valid for up to 7 days. Acceptable for a low-risk social app where re-login friction is the bigger concern.

### Secure Storage

Raw JWTs must never be stored in `localStorage` (XSS-accessible). Mobile storage:
- **Expo SecureStore** (SBCA) — backed by iOS Keychain / Android Keystore; hardware-protected on supported devices
- **iOS Keychain** (Whosup SwiftUI) — native secure enclave storage

### Stateless Trade-offs

| Pro | Con |
|-----|-----|
| No DB lookup per request — fast | Cannot invalidate individual tokens before expiry |
| Scales horizontally — no shared session store | Token payload is readable (base64 decoded) — never store secrets in it |
| Works offline for validation | Secret key rotation invalidates all existing tokens |

Immediate revocation (e.g., on logout or account ban) requires a token denylist, which re-introduces a DB lookup and removes the stateless benefit. Both SBCA and Whosup accept the trade-off for simplicity at their scale.

## Relationships to Other Concepts

- [[wiki/concepts/oauth2-proxy-pattern]] — alternative auth strategy: proxy intercepts at the network layer; backend never handles auth
- [[wiki/concepts/webhook-patterns]] — webhook endpoints are unauthenticated; JWT doesn't apply there

## External References

- JWT specification: https://jwt.io/introduction
