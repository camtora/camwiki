---
title: "Quartz Wiki"
type: infrastructure
tags: [infra, wiki, quartz, nodejs]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# Quartz Wiki

Quartz v4 static site generator serving the camwiki knowledge base at `wiki.camerontora.ca`. Protected by Google OAuth2.

## Specification

| Property | Value |
|----------|-------|
| URL | `https://wiki.camerontora.ca` |
| Port | 3005 |
| Install path | `/home/camerontora/quartz-wiki` |
| Content | Symlink: `quartz-wiki/content` → `/home/camerontora/camwiki` |
| Node version | 22 (via nvm: `/home/camerontora/.nvm/versions/node/v22.22.2`) |
| Auth | OAuth2 (all authenticated users) |
| Systemd service | `quartz-wiki` |

## Role in the Stack

Nginx proxies `wiki.camerontora.ca` → `http://host.docker.internal:3004`. Quartz serves the built static site and watches for content changes. The content symlink means edits to the camwiki repo are picked up automatically without any manual sync step.

## Managing the Service

```bash
sudo systemctl status quartz-wiki     # check status
sudo systemctl restart quartz-wiki    # restart after config changes
sudo journalctl -u quartz-wiki -f     # tail logs
```

## Rebuilding

Quartz `serve` mode watches for content changes and rebuilds automatically. To force a full rebuild:

```bash
cd /home/camerontora/quartz-wiki
source ~/.nvm/nvm.sh && nvm use 22
npx quartz build
sudo systemctl restart quartz-wiki
```

## Configuration

`/home/camerontora/quartz-wiki/quartz.config.ts`:
- `baseUrl: "wiki.camerontora.ca"`
- `ignorePatterns: ["raw", "scripts", ".obsidian", ".claude", ".git", "**/*.sh", "**/*.py"]`
- Analytics disabled

## Known Issues / Limitations

- Node 22 required (Quartz 4.5.2 minimum). Node version managed via nvm — systemd service uses hardcoded path `/home/camerontora/.nvm/versions/node/v22.22.2/bin/npx`. Update this path if Node 22 is upgraded.
- `raw/` directory excluded from build via ignorePatterns — contains symlinks to full git repos that would slow the build significantly.
- Git date warnings for files not tracked by camwiki git repo are harmless — dates fall back to modification time.

## Connected Projects

- [[wiki/projects/camwiki]]
- [[wiki/infrastructure/nginx-reverse-proxy]]
- [[wiki/infrastructure/oauth2-proxy]]
