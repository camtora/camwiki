---
title: "Storage and RAID"
type: infrastructure
tags: [infra, storage, raid, nas, smart]
created: 2026-04-16
updated: 2026-04-17
source_count: 2
---

# Storage and RAID

Two RAID arrays provide storage on the home server. Both are monitored via the [[wiki/infrastructure/health-api]] and surfaced in the [[wiki/infrastructure/status-dashboard]].

## HOMENAS (Primary NAS)

| Property | Value |
|----------|-------|
| Device | `md1` |
| RAID UUID | `1a922a4c-a4f5-ee48-e889-04e17cbd42d1` |
| Type | Linux software RAID5 |
| Drives | 10 drives (sdc–sdl) — see drive table below |
| Mount | `/HOMENAS` |
| Health source | `/proc/mdstat` |
| macOS SSHFS | Mounts as `HOMENAS` in Finder |

### HOMENAS Drive Health (as of 2026-04-17 snapshot)

| Device | UUID sub | Age | SMART Status |
|--------|----------|-----|--------------|
| sdc | f0515eaf | 3y 6m 23d | OK — one attribute failed in the past |
| sdd | 4bc3dc1e | 2y 9m 6d | OK — one attribute failed in the past |
| sde | 1daf6693 | 2y 6m 15d | **OK — 16 bad sectors** ⚠️ replacement candidate |
| sdf | d96af91f | 3y 6m 23d | OK — one attribute failed in the past |
| sdg | c0a09f2f | — | No SMART data available |
| sdh | 0870b08a | — | No SMART data available |
| sdi | 4c6c1421 | — | No SMART data available |
| sdj | 6b35dc01 | — | No SMART data available |
| sdk | 8478ad0d | — | No SMART data available |
| sdl | b8e23ae1 | — | No SMART data available |

**sde is the priority replacement candidate** — 16 bad sectors in a RAID5 array means one more drive failure would be catastrophic.

### Monitoring

- RAID status (active/degraded/failed) and sync progress parsed from `/proc/mdstat`
- Per-drive SMART data via `smartctl`: temperature, power-on hours, SMART status, reallocated sectors, pending sectors, uncorrectable errors
- Dashboard shows expandable drive list with individual health per drive
- Warnings triggered if any drive has non-zero sector counts

## CAMRAID

| Property | Value |
|----------|-------|
| Device | `sdk` |
| Type | Hardware RAID5 |
| Mount | CAMRAID (macOS SSHFS mounts this in Finder) |
| Health source | Mount status check |

## CAMNAS2-Labeled Drives (Likely HOMENAS2)

Six drives labelled CAMNAS2-1 through CAMNAS2-6 appear in the drive snapshot. These are separate from the HOMENAS RAID members and likely form the `/HOMENAS2` secondary mount.

| Label | Device | UUID |
|-------|--------|------|
| CAMNAS2-1 | sdb | 78587b9c-f4ba-4ba3-9c94-611ff1c68f2a |
| CAMNAS2-2 | sdm | 7abe2ff9-576f-402e-9b9c-e9bd5fe039b2 |
| CAMNAS2-3 | sdn | e8310705-e937-4b88-a084-f708df0977f6 |
| CAMNAS2-4 | sdl | 2433cb0b-6f03-4f9c-b75c-794ef25a91ee |
| CAMNAS2-5 | sdk | 3095ac79-242c-4999-9e78-682bcbea3309 |
| CAMNAS2-6 | sdl | b2be79ea-9277-47d8-b4aa-12c7dbd7bb5b |

Note: sdl and sdk appear in both the HOMENAS RAID table and this table — likely reflects a drive reassignment. Verify current physical mapping with `lsblk`.

## System Drive

| Device | Partition | Mount |
|--------|-----------|-------|
| sda1 | EFI System (vfat) | EFI |
| sda2 | ext4 | `/` |
| sda3 | swap | swap |
| sda4 | ext4 | `/tmp` |
| sda5 | ext4 | `/var` |
| sda6 | ext4 | `/home` |

Age: 3y 7m 28d — SMART status OK.

## Open Maintenance TODOs (from snapshot)

- Fan curves need configuring
- Replace rear fan
- Evaluate swapping sda/sdb — end of life?
- Plex metadata volumes issue in Docker (user library possibly under plugins and not saved)
- TV show and talk/reality show metadata refresh needed
- Reverse-sync Tautulli history to Plex (or via Trakt)
- 503 error page when services are down

## macOS SSHFS Access

A script (`infrastructure/scripts/`) mounts both arrays in macOS Finder via SSHFS. Compatible with macOS bash 3.2.

## Reboot Verification

After a server reboot, the [[wiki/infrastructure/status-dashboard]] admin panel polls for both HOMENAS and CAMRAID to confirm healthy and mounted before marking the reboot as complete.

## Connected Projects

- [[wiki/infrastructure/health-api]]
- [[wiki/infrastructure/status-dashboard]]
- [[wiki/infrastructure/home-server]]

## Sources

- [[wiki/sources/infrastructure-repo]] — RAID monitoring, SMART data, SSHFS mount script, reboot verification
- [[wiki/sources/desktop-hard-drive-arrangement]] — per-drive UUID and SMART health snapshot, CAMNAS2 drive labels, open maintenance TODOs
