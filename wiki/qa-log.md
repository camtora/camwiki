---
title: "Q&A Log"
type: concept
tags: [qa, log]
created: 2026-04-17
updated: 2026-04-17
---

# Q&A Log

Append-only record of questions asked to the wiki during Claude Code sessions.
Never edit past entries — always append at the bottom.

---

## 2026-04-17 — GoDaddy DDNS script location

**Q:** Where is the GoDaddy update script documented? Or the cron jobs that run the speed tests/health checks?

**A:** Both are documented. GoDaddy DDNS script (`infrastructure/scripts/godaddy-ddns.sh`) is in [[wiki/infrastructure/dns-ssl]] — cron at `/etc/cron.d/godaddy-ddns` every 10 min, log at `/var/log/godaddy-ddns.log`. Speed test cron (`infrastructure/scripts/speedtest.sh`) is in [[wiki/infrastructure/health-api]] — cron at `/etc/cron.d/speedtest` every 30 min, output at `/var/lib/health-api/speedtest.json`.

---

## 2026-04-17 — All scheduled/automated tasks

**Q:** I think it would help to have everything that automatically runs in one place.

**A:** Created [[wiki/concepts/scheduled-tasks]] — covers 6 home server crons (DDNS, speedtest, VNC, wiki upload, location backfill, Haymaker DB backup), 2 GCP Cloud Scheduler jobs, 3 in-process background threads, 4 app-level scheduled jobs (Recyclarr, SBCA node-cron, Haymaker reminders), and frontend polling intervals.

_New page created: [[wiki/concepts/scheduled-tasks]]_

---
