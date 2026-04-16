---
title: "Source: camerontora.ca repo"
type: source
tags: [source, website, nextjs, plex, map, oauth]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Source: camerontora.ca repo

**Raw file:** `raw/repos/camerontora.ca`
**Ingested:** 2026-04-16
**Type:** repo

## Summary

The camerontora.ca repo is the personal landing page and home dashboard. Built with Next.js 15 + React 19 + Tailwind CSS 4, deployed via Docker Compose. The main feature is an interactive Plex watch history map (Leaflet + CARTO dark basemap) showing geolocated Tautulli stream history, with live activity markers updated in real time.

The site has its own OAuth2 Proxy instance (port 4182) separate from the infrastructure stack's shared proxy. Authentication uses Google SSO; authorized users see private service tiles for Haymaker, SBCA, backend tools, and other personal projects.

A key cross-repo dependency: the Watchmap backend writes location data to `camerontora.ca/app/data/locations.json`, which the camerontora.ca API reads. This file is bind-mounted read-only into the Watchmap container.

## Key Facts / Data Points

- Stack: Next.js 15, React 19, Tailwind CSS 4, Leaflet, Docker Compose
- Internal architecture: Apache (443) → OAuth2 Proxy (4182) → Next.js (3002)
- Map data stored in `app/data/locations.json` (gitignored for privacy); backfilled hourly via cron (`0 * * * *`)
- Geolocation via ip-api.com
- Auth: `x-forwarded-email` header checked against `allowedEmails` array in `page.tsx`
- Service tile categories: flagship (SBCA, Who's Up), personal projects (Haymaker, Rotosync), backend (Radarr, Sonarr, etc.)
- Status banner severity pulled from status API (major/minor/degraded/healthy)
- Contact form + Plex access request form both post to Discord webhook
- `data/locations.json` is a shared file: Watchmap writes it, camerontora.ca reads it

## Relevance to Wiki

Documents the public-facing website and its role as a home dashboard. Reveals a cross-repo data dependency between Watchmap and camerontora.ca not documented elsewhere.

## Contradictions or Tensions

- camerontora.ca has its own OAuth2 Proxy instance (port 4182) — separate from the shared infrastructure OAuth2 Proxy. Updated [[wiki/infrastructure/oauth2-proxy]] to note both instances.

## Pages Updated

- [[wiki/projects/camerontora-ca]] — created
- [[wiki/infrastructure/oauth2-proxy]] — noted second instance in camerontora.ca stack
