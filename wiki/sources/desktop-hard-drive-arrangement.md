---
title: "Source: Hard Drive Arrangement"
type: source
tags: [source, infra, storage, raid, smart]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# Source: Hard Drive Arrangement

**Raw file:** `~/Desktop/hard_drive_arrangement.txt`
**Ingested:** 2026-04-17
**Type:** doc

## Summary

Drive layout snapshot for the CAMNAS2 home server. Lists all block devices with their UUIDs, RAID membership, SMART health summary, and age. System drive sda has 5 partitions (EFI, root, swap, tmp, var, home). The HOMENAS software RAID5 spans 10 drives (sdc–sdl), and a second set of 6 drives are labelled CAMNAS2-1 through CAMNAS2-6 (likely HOMENAS2 or a separate array).

SMART status reveals several aging drives: sde has 16 bad sectors and should be monitored for replacement. sdc, sdd, and sdf all have "one attribute failed in the past." sdg through sdl show "No data available," indicating SMART is not yet configured or reporting for those drives.

The file also contains a maintenance TODO list (fan curves, drive swaps, Plex metadata issues, Tautulli reverse sync, 503 error page).

## Key Facts / Data Points

- Server hostname: CAMNAS2
- System drive: `/dev/sda` — age 3y 7m 28d, health OK
- HOMENAS RAID label: `CAM-NAS:0` (Linux software RAID5)
- `/dev/sde`: **16 bad sectors** — priority replacement candidate
- `/dev/sdc`, `/dev/sdd`, `/dev/sdf`: "one attribute failed in the past"
- `/dev/sdg`–`/dev/sdl`: "No data available" from SMART
- CAMNAS2-labeled drives: sdb(CAMNAS2-1), sdm(CAMNAS2-2), sdn(CAMNAS2-3), sdl(CAMNAS2-4), sdk(CAMNAS2-5), sdl(CAMNAS2-6)
- Note: sdl and sdk appear in both HOMENAS and CAMNAS2 drive lists — likely reflects reassignment or a snapshot from different times. Verify with `lsblk` for current state.

## Relevance to Wiki

- [[wiki/infrastructure/storage-raid]] — drive-level SMART data and CAMNAS2-labeled drives

## Contradictions or Tensions

- storage-raid.md says HOMENAS has 8 drives; this file lists 10 entries under CAM-NAS:0. Could reflect expansion over time.
- sdl and sdk appear in both HOMENAS RAID and CAMNAS2 drive tables — likely stale data; current physical mapping should be confirmed with `lsblk`.

## Pages Updated

- [[wiki/infrastructure/storage-raid]] — per-drive SMART status, open maintenance TODOs
