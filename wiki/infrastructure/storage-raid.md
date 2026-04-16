---
title: "Storage and RAID"
type: infrastructure
tags: [infra, storage, raid, nas, smart]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# Storage and RAID

Two RAID arrays provide storage on the home server. Both are monitored via the [[wiki/infrastructure/health-api]] and surfaced in the [[wiki/infrastructure/status-dashboard]].

## HOMENAS (Primary NAS)

| Property | Value |
|----------|-------|
| Device | `md1` |
| Type | Linux software RAID5 |
| Drives | 8 drives (`sdc`–`sdj`) |
| Capacity | ~100 TB (from commit history: CAMNAS2 added for 100TB RAID) |
| Mount | HOMENAS (macOS SSHFS mounts this as `HOMENAS` in Finder) |
| Health source | `/proc/mdstat` |

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
