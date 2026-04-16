---
title: "macOS Remote Mount (SSHFS)"
type: infrastructure
tags: [infra, macos, sshfs, storage, remote-access]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# macOS Remote Mount (SSHFS)

MacBook mounts the home server's storage over SSH using SSHFS. Samba is disabled — SSHFS is the only remote filesystem method. Works both on the home network and remotely.

## Specification

| Property | Value |
|----------|-------|
| Method | SSHFS (macFUSE) |
| Internal | `192.168.2.34:22` (home network) |
| External | `camerontora.ca:2222` (away from home) |
| Scripts | `infrastructure/scripts/macos/` |

## Mount Points (on MacBook)

| Mount Point | Remote Path | Content |
|-------------|-------------|---------|
| `~/mnt/HOMENAS` | `/home/camerontora` | Home directory |
| `~/mnt/CAMNAS2` | `/HOMENAS` | 100 TB software RAID (Plex media) |
| `~/mnt/CAMRAID` | `/CAMRAID` | Hardware RAID (personal media) |

## Scripts

| File | Purpose |
|------|---------|
| `scripts/macos/mount-camerontora.sh` | Smart mount — auto-detects home vs external network |
| `scripts/macos/com.camerontora.sshfs-mount.plist` | launchd automount at login |

Installed on MacBook at:
- `/opt/homebrew/bin/mount-camerontora`
- `~/Library/LaunchAgents/com.camerontora.sshfs-mount.plist`

## Commands

```bash
mount-camerontora              # mount all three
mount-camerontora unmount      # unmount all
mount-camerontora status       # check status
mount-camerontora remount      # remount after network change

mount-camerontora mount home   # home directory only
mount-camerontora mount media  # 100TB RAID only
mount-camerontora mount raid   # hardware RAID only
```

## Network Detection

Script pings `192.168.2.34` to detect if on the home network, then selects the appropriate SSH host and port. SSH keys stored in macOS keychain (`ssh-add --apple-use-keychain`).

## Automount

The launchd plist runs at login and mounts all three. After a network change during a session (leaving/arriving home), mounts go stale and require a manual `mount-camerontora remount`.

## Prerequisites (MacBook Setup)

```bash
brew install macfuse             # requires reboot + Security & Privacy approval
brew install gromgit/fuse/sshfs-mac
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
mkdir -p ~/mnt/HOMENAS ~/mnt/CAMNAS2 ~/mnt/CAMRAID
```

## Troubleshooting

```bash
# Check if mounted
mount | grep HOMENAS

# View automount logs
cat /tmp/sshfs-mount.log

# Force unmount if stuck
diskutil unmount force ~/mnt/HOMENAS

# Load SSH keys from keychain
ssh-add --apple-load-keychain

# Test SSH connectivity
ssh -p 2222 camerontora@camerontora.ca "echo connected"
```

## Known Issues / Limitations

- No auto-reconnect after network change — manual `remount` required
- macFUSE requires Security & Privacy approval on each macOS major upgrade
- Automount only fires at login, not on network change

## Connected Projects

- [[wiki/infrastructure/home-server]]
- [[wiki/infrastructure/storage-raid]]

## Sources

- [[wiki/sources/infrastructure-repo]] — MACOS-MOUNT.md, mount scripts
