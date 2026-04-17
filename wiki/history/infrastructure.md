---
title: "History: Infrastructure"
type: project
tags: [history, infrastructure]
created: 2026-04-17
updated: 2026-04-17
source_count: 1
---

# History: Infrastructure

_Generated from `git log` as of 2026-04-17. Re-generated on every repo ingest._

| Hash | Date | Message |
|------|------|---------|
| `1e16fa0` | 2026-04-17 | Fix deploy.sh to derive --set-secrets from REQUIRED_SECRETS list |
| `b109fc6` | 2026-04-17 | Add public Wiki Q&A panel to status dashboard |
| `e59329c` | 2026-04-17 | Add Wiki Q&A feature spec for status dashboard |
| `4e2a447` | 2026-04-16 | Clarify Plex banner: issue is with Plex the company, not our server |
| `3d6a4d9` | 2026-04-16 | Document Plex platform banner incident detail display |
| `7b813ed` | 2026-04-16 | Improve Plex status banner: show incident name, status, and link |
| `4d65582` | 2026-04-16 | Add Plex platform status banner and grant jshor96 OAuth access |
| `4bad08b` | 2026-03-28 | Document ombi decommission and updated DNS failover records |
| `1869a57` | 2026-03-28 | Remove ombi from DNS failover — ombi page stays on home server only |
| `1e065b3` | 2026-03-28 | Restore ombi to DNS failover records |
| `8eb6890` | 2026-03-28 | Remove Ombi from status dashboard monitoring |
| `ffdb0d2` | 2026-03-28 | Fix Seerr missing container stats on status dashboard |
| `d98dc7b` | 2026-03-28 | Update Ombi landing page: retired, redirect to Seerr |
| `33cbb99` | 2026-03-27 | Add seerr.camerontora.ca; redirect overseerr → seerr (Phase 1) |
| `2ce6d3c` | 2026-03-11 | Add metro.sba subdomain; switch to *.sba wildcard DNS |
| `2cec355` | 2026-03-10 | Add Netdata /var disk alarm with 75% warning threshold |
| `e438431` | 2026-03-08 | Add sba and admin.sba subdomains to cert, DDNS, and nginx |
| `803ccdc` | 2026-03-07 | Add SBA backend requirements doc; update Transmission queue notes |
| `2b531c3` | 2026-03-07 | Add sba.camerontora.ca subdomain infrastructure |
| `c2b631a` | 2026-02-23 | Fix Transmission: queue, port forwarding, and VPN grace period |
| `fb94f11` | 2026-02-20 | Close per-service authorization backlog item |
| `a033a55` | 2026-02-20 | Close Haymaker health endpoint backlog item |
| `57e25a8` | 2026-02-20 | Mark DNS-FAILOVER.md test checklist as complete |
| `64f9b17` | 2026-02-20 | Mark DNS Failover as completed |
| `834b5c5` | 2026-02-20 | Add DNS failover documentation and update backlog |
| `90eecc2` | 2026-02-20 | Implement DNS failover protection and expand to public subdomains |
| `3bd2fa9` | 2026-02-06 | Add /uploads/ location to whosup nginx config |
| `e4800f3` | 2026-02-06 | Strip /api prefix from whosup proxy requests |
| `89877ab` | 2026-02-06 | Fix false mount failure reports in SSHFS mount script |
| `39e5da3` | 2026-01-31 | Update WHOSUP.md: replace Uptime Kuma with status dashboard |
| `061962b` | 2026-01-31 | Add Who's Up to status dashboard and update docs |
| `108df5c` | 2026-01-31 | Add CAMRAID mount to macOS SSHFS script |
| `c780658` | 2026-01-31 | Fix mount script for macOS bash 3.2 compatibility |
| `5338ff6` | 2026-01-31 | Add CAMNAS2 mount for 100TB RAID |
| `625abc9` | 2026-01-31 | Update Who's Up project plan and add to DDNS |
| `2199658` | 2026-01-31 | Add Who's Up infrastructure (whosup.camerontora.ca) |
| `ae3d0a7` | 2026-01-28 | Change SSHFS volume name to HOMENAS in Finder |
| `1858cc5` | 2026-01-27 | Fix sshfs path for launchd (use full path) |
| `fb5055c` | 2026-01-27 | Fix mount point for macOS (~/mnt/HOMENAS instead of /mnt) |
| `96e3a4b` | 2026-01-27 | Update mount scripts for Apple Silicon (/opt/homebrew/bin) |
| `facbd19` | 2026-01-27 | Add macOS SSHFS mount scripts for remote file access |
| `0aef305` | 2026-01-27 | Add Feb 1 action plan for GoDaddy API restoration |
| `fe3f0f5` | 2026-01-27 | Remove dholley.swp@gmail.com from oauth2-proxy allowlist |
| `c34d45d` | 2026-01-20 | Improve reboot dialog UX with state machine tracking |
| `d974c06` | 2026-01-19 | Add public endpoint for Apple Health webhooks |
| `84b1b76` | 2026-01-18 | Update DNS failover explanation and remove stale monitor subdomain |
| `9d2d8a7` | 2026-01-18 | Add explanatory text to DNS Configuration panel |
| `133fc55` | 2026-01-18 | Add emergency API key authentication for admin access |
| `c807c80` | 2026-01-18 | Add Login button to header when not authenticated |
| `8645cd4` | 2026-01-18 | Update page title to Status \| CAM TORA |
| `89f348b` | 2026-01-18 | Keep reboot dialog open showing services, add Close button when done |
| `84e33e2` | 2026-01-18 | Fix status dashboard resilience when home server is down |
| `55d04d8` | 2026-01-18 | Update docs to reflect nsenter reboot mechanism |
| `1bd8875` | 2026-01-18 | Use nsenter for host reboot, switch Transmission to Montreal VPN |
| `14ec41e` | 2026-01-17 | Update Transmission proxy to Toronto VPN (port 9091) |
| `6bc8571` | 2026-01-17 | Add dholley.swp@gmail.com to authenticated users |
| `dcb945e` | 2026-01-17 | Add VNC session monitor to fix RealVNC sticky cloud sessions |
| `2dff7e4` | 2026-01-17 | Add public endpoint for Withings OAuth callback |
| `c8cf42c` | 2026-01-16 | Fix VPN switch to update Sonarr/Radarr after Transmission is ready |
| `5a54a5a` | 2026-01-14 | Document auto-repair connectivity check fix (Issue #10) |
| `2c215e4` | 2026-01-14 | Add real connectivity check and improve auto-repair |
| `287ba7b` | 2026-01-14 | Document gluetun internal DNS initialization issue |
| `0f63e20` | 2026-01-14 | Document gluetun memory leak and restart loop fixes |
| `dbb07ab` | 2026-01-14 | Fix transmission orphaning and API key issues |
| `3e877e1` | 2026-01-14 | Add VPN switch troubleshooting guide |
| `9e64437` | 2026-01-14 | Improve VPN switch reliability and status dashboard health checks |
| `f02025e` | 2026-01-14 | Switch Transmission VPN from Vancouver to Montreal |
| `17a2e66` | 2026-01-14 | Fix VPN switch failing due to incompatible --dns flag |
| `4817d90` | 2026-01-14 | Add auto-repair for orphaned transmission containers |
| `8dd4970` | 2026-01-13 | Document alert severity levels and VPN alerting behavior |
| `0f22ba4` | 2026-01-13 | Change VPN failover alerts from major to minor severity |
| `2d625e8` | 2026-01-13 | Only alert when active VPN is unhealthy |
| `093a36c` | 2026-01-13 | Add DNS Failover to blocked items in backlog |
| `97705a8` | 2026-01-13 | Add VPN auto-failover (30 min threshold) |
| `1b3107f` | 2026-01-13 | Fix Transmission port check for dynamic VPN |
| `653d030` | 2026-01-13 | Add outage severity levels (major/minor/degraded) |
| `8c817ed` | 2026-01-13 | Document nginx http2 deprecation fix in backlog |
| `77435ee` | 2026-01-13 | Fix nginx http2 deprecation warnings |
| `fad9390` | 2026-01-13 | Document Jellyfin/Emby decision: inactive, DNS removed |
| `d2e1811` | 2026-01-13 | Document authorization and custom 403 error page in SSO guide |
| `7130a57` | 2026-01-13 | Update 403 page messaging |
| `eda3bb9` | 2026-01-13 | Fix oauth2-proxy authorization and custom 403 page |
| `42702f9` | 2026-01-13 | Document custom 403 error page completion in backlog |
| `6442a93` | 2026-01-13 | Add custom 403 error page for unauthorized users |
| `becc0c7` | 2026-01-13 | Add SMART disk health monitoring for RAID drives |
| `973d495` | 2026-01-13 | Fix CORS origin for admin endpoints |
| `0fa4348` | 2026-01-13 | Add RAID array health monitoring to status dashboard |
| `4f8ae74` | 2026-01-13 | Add server reboot feature documentation |
| `940461f` | 2026-01-13 | Update backlog: SSH Restart Server completed |
| `911fa69` | 2026-01-13 | Add server restart feature to status dashboard |
| `a8aea6b` | 2026-01-13 | Migrate status dashboard from monitor.camerontora.ca to status.camerontora.ca |
| `58eb48a` | 2026-01-13 | Update Sonarr/Radarr ports automatically on VPN switch |
| `29f5c91` | 2026-01-13 | Update backlog: Discord alerts and GitHub Actions fixes complete |
| `8733ed4` | 2026-01-13 | Fix speed test field name: upload_mbps -> upload |
| `0aac531` | 2026-01-13 | Fix VPN switch to update all nginx proxy_pass lines |
| `9c389f8` | 2026-01-13 | Fix service account creation error handling |
| `46965aa` | 2026-01-13 | Add GCP auth setup script for GitHub Actions |
| `0e2e19e` | 2026-01-13 | Add GitHub Actions workflow for status-dashboard deployment |
| `5b1508c` | 2026-01-13 | Run health checks concurrently to avoid timeout cascade |
| `4e613e2` | 2026-01-13 | Revert admin-only access - keep health endpoints |
| `acc75da` | 2026-01-12 | Run health checks concurrently (prevents timeout cascade) |
| `8fb24e7` | 2026-01-12 | Fix redirect handling in health checker |
| `f53c651` | 2026-01-12 | Add unauthenticated health endpoints for external monitoring |
| `fbb2674` | 2026-01-12 | Add per-service authorization for admin-only access |
| `965cd82` | 2026-01-12 | Update speed test docs: 5-min interval, concurrent tests |
| `4e17b04` | 2026-01-12 | Run VPN speedtests concurrently (3x faster) |
| `a404ac7` | 2026-01-12 | Document container restart and uptime display features |
| `50a67b9` | 2026-01-12 | Spin until uptime drops (status refresh shows restart) |
| `69be942` | 2026-01-12 | Revert to simple restart - 5 second spinner feedback |
| `d50a0fc` | 2026-01-12 | Poll until uptime visually updates after restart |
| `a40f1a7` | 2026-01-12 | Spin until status refresh completes |
| `4e401fd` | 2026-01-12 | Move uptime under response time, green for 5min after restart |
| `3347fb5` | 2026-01-12 | Fix: pass container_uptime through run_health_check merge |
| `a11bd3c` | 2026-01-12 | Fix requirements.txt - revert to working version |
| `d846560` | 2026-01-12 | Fix requirements.txt - bump firestore version to force rebuild |
| `ad84acc` | 2026-01-12 | Force rebuild |
| `d169153` | 2026-01-12 | Add container uptime display to status cards |
| `e9d59b6` | 2026-01-12 | Keep restart spinner visible for at least 2 seconds for feedback |
| `32b1676` | 2026-01-12 | Make container restart async - return immediately, restart in background |
| `5ca268b` | 2026-01-12 | Add 60 second timeout to container restart fetch |
| `fade435` | 2026-01-12 | Add type=button to restart, improve error logging |
| `4d81c84` | 2026-01-12 | Add longer timeout for admin endpoints (container restart) |
| `28a58cd` | 2026-01-12 | Fix admin email default to cameron.tora@gmail.com |
| `d2c694a` | 2026-01-12 | Fix admin email in docs |
| `536a789` | 2026-01-12 | Add container restart from dashboard |
| `79fd58f` | 2026-01-12 | Add SSL certificate expiry monitoring to gcp-monitor |
| `29482db` | 2026-01-12 | Clean up DNS panel for non-admin view |
| `59eb269` | 2026-01-12 | Simplify DNS panel: use OAuth auth, remove test mode |
| `2bf0eb0` | 2026-01-12 | Add VPN health alerting to gcp-monitor |
| `7b42b77` | 2026-01-12 | Return mock DNS data when GoDaddy API unavailable |
| `9cc8369` | 2026-01-12 | Redesign DNS panel with two-card layout and test mode |
| `41eaadf` | 2026-01-12 | Add .gitignore for frontend node_modules and dist |
| `4e503b7` | 2026-01-12 | Replace confirm dialog with inline confirmation button |
| `f9e466c` | 2026-01-12 | Improve VPN switch UX: spinner until active, remove refresh button |
| `8ab16a1` | 2026-01-12 | Auto-update speedtest.json active status after VPN switch |
| `8f18e70` | 2026-01-12 | Add admin panel with VPN switching functionality |
| `e24271c` | 2026-01-12 | Move VPN Switch button to speed results line |
| `438bcf9` | 2026-01-12 | Fix VPN health check to use speedtest data instead of Docker health |
| `d739886` | 2026-01-12 | Integrate VPN switching into SpeedPanel |
| `370230a` | 2026-01-12 | Add admin panel with VPN location switching |
| `d912c5f` | 2026-01-12 | Widen Ombi migration page container (480px → 560px) |
| `801367c` | 2026-01-12 | Soften thank you note on Ombi migration page |
| `50e1b38` | 2026-01-12 | Remove What's New section from Ombi migration page |
| `2236e70` | 2026-01-12 | Update Ombi migration page |
| `97fce9a` | 2026-01-12 | Update Ombi migration page with personal tone and mobile instructions |
| `9a757dc` | 2026-01-12 | Document real-time metrics feature in STATUS-DASHBOARD.md |
| `63245ab` | 2026-01-12 | Use 10-second rolling average for CPU/RAM metrics |
| `3cf33b4` | 2026-01-12 | Fix duplicate CORS header on Netdata metrics endpoints |
| `f630087` | 2026-01-12 | Add real-time CPU/RAM metrics from Netdata |
| `85265b0` | 2026-01-12 | Remove self-referential monitor checks from GCP services |
| `87d6200` | 2026-01-12 | Decommission Uptime Kuma, redirect status → monitor |
| `83b24db` | 2026-01-12 | Switch SSL cert renewal from Apache to webroot mode |
| `97fd31b` | 2026-01-11 | Update STATUS-DASHBOARD.md with recent changes |
| `a3294b3` | 2026-01-11 | Update title to CAM TORA \| Status, update Phase 4 plan |
| `12962a0` | 2026-01-11 | Restyle dashboard + fix Firestore project initialization |
| `9c75320` | 2026-01-11 | Add /tmp and /dev (RAM) volume monitoring, fix VPN active detection |
| `c367d13` | 2026-01-11 | Migrate Transmission to Toronto VPN, show all VPN locations on dashboard |
| `d7b54c5` | 2026-01-11 | Document Phase 3 historical data feature |
| `e83fc48` | 2026-01-11 | Phase 3: Add historical uptime tracking |
| `c2127e1` | 2026-01-11 | Document internal vs external health checks |
| `ab93cce` | 2026-01-11 | Add internal service status to dashboard UI |
| `b827e3c` | 2026-01-11 | Add internal service checks to health-api |
| `2cb9882` | 2026-01-11 | Document health check logic in STATUS-DASHBOARD.md |
| `33c04d6` | 2026-01-11 | Update docs with DNS failover details |
| `1a9d879` | 2026-01-11 | Limit DNS failover to @ and monitor only |
| `300dda2` | 2026-01-11 | Add DNS failover with dynamic home IP detection |
| `65978f2` | 2026-01-11 | Speedtest all active VPN locations and display in dashboard |
| `59cc71f` | 2026-01-11 | Use Vancouver VPN instead of Toronto for speedtest |
| `f9d711d` | 2026-01-11 | Add VPN speedtest through gluetun-toronto |
| `c781a6f` | 2026-01-11 | Update speedtest to use local Ookla CLI instead of Docker |
| `10e5d54` | 2026-01-11 | Fix speedtest.sh: redirect log output to stderr to prevent JSON corruption |
| `98d55db` | 2026-01-11 | Fix disk field names, DNS error handling, and add last update timestamp |
| `6000763` | 2026-01-11 | Fix static folder path and reduce gunicorn workers for Cloud Run |
| `1ff295a` | 2026-01-11 | Fix Python import paths for Docker deployment |
| `a921491` | 2026-01-11 | Fix Tailwind CSS: replace non-existent bg-gray-750 with bg-gray-700 |
| `4041405` | 2026-01-11 | Add GCP status dashboard for monitoring camerontora.ca services |
| `3a903c7` | 2026-01-11 | Pass OAuth access token to enable profile photo/name |
| `86ef7b8` | 2026-01-11 | Remove pycache and add gitignore |
| `7f9609a` | 2026-01-11 | Add external monitoring via GCP Cloud Run |
| `ce16ec4` | 2026-01-11 | Optimize GoDaddy DDNS script with batch API calls |
| `60e7209` | 2026-01-11 | Update README with Netdata, Uptime Kuma, and docs references |
| `f476470` | 2026-01-11 | Document nginx-proxy stop/start for certbot |
| `6bd020d` | 2026-01-11 | Add Netdata monitoring and consolidate Uptime Kuma |
| `a5abc97` | 2026-01-11 | Update Transmission proxy to Vancouver VPN port (9093) |
| `51b0472` | 2026-01-10 | Update VPN documentation for WireGuard |
| `38597c1` | 2026-01-10 | Add Ombi migration page and update security docs |
| `ef412aa` | 2026-01-10 | Add security documentation |
| `b572657` | 2026-01-10 | Initial infrastructure setup with unified SSO |
