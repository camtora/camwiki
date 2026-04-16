---
title: "DNS and SSL"
type: infrastructure
tags: [infra, dns, ssl, letsencrypt, godaddy, ddns]
created: 2026-04-16
updated: 2026-04-16
source_count: 1
---

# DNS and SSL

GoDaddy Dynamic DNS keeps all subdomains pointed at the home server's public IP. Let's Encrypt issues a shared SSL certificate for all subdomains.

## Dynamic DNS (GoDaddy)

| Property | Value |
|----------|-------|
| Registrar | GoDaddy |
| Script | `infrastructure/scripts/godaddy-ddns.sh` |
| Credentials | `/etc/godaddy-ddns.env` (chmod 600) |
| Cron | `/etc/cron.d/godaddy-ddns`, every 10 minutes |
| API calls | ~8,640/month (well under 20k limit) |
| Log | `/var/log/godaddy-ddns.log` |

### Managed A Records

`@`, `plex`, `sonarr`, `radarr`, `tautulli`, `transmission`, `jackett`, `overseerr`, `seerr`, `watchmap`, `haymaker`, `netdata`, `health`, `whosup`, `sba`, `*.sba`, `ombi` (decommissioned, DNS kept live for migration page)

### DNS Failover Sentinel

The DDNS script checks whether `@` currently points to the GCP static IP (`216.239.32.21`). If so, it skips the update — preventing the cron from undoing an active failover. See [[wiki/concepts/dns-failover]].

### Adding a Subdomain

1. Add to `RECORDS` array in `scripts/godaddy-ddns.sh`
2. Run manually: `sudo /home/camerontora/infrastructure/scripts/godaddy-ddns.sh`
3. Add to SSL cert (see below)
4. Add nginx config

## SSL Certificate (Let's Encrypt)

| Property | Value |
|----------|-------|
| Cert name | `camerontora-services` |
| Location | `/etc/letsencrypt/live/camerontora-services/` |
| Method | Webroot (nginx stays running during renewal) |
| Webroot | `/var/www/acme` |
| Renewal | Automatic via systemd certbot.timer |

### Current Domains (19)

`camerontora.ca`, `www.camerontora.ca`, `haymaker`, `health`, `jackett`, `netdata`, `ombi`, `overseerr`, `seerr`, `plex`, `radarr`, `sonarr`, `tautulli`, `transmission`, `watchmap`, `whosup`, `sba`, `admin.sba`, `metro.sba` (all `.camerontora.ca`)

`status.camerontora.ca` is a CNAME to GCP Cloud Run — SSL managed by Google, not in this cert.

### Adding a Subdomain to the Certificate

Must list ALL existing domains or missing ones get removed from the cert.

```bash
sudo certbot certonly --webroot \
  -w /var/www/acme \
  --cert-name camerontora-services \
  -d camerontora.ca \
  # ... all existing domains ... \
  -d NEW_SUBDOMAIN.camerontora.ca

docker exec nginx-proxy nginx -s reload
```

### ACME Gotcha

`00-http-redirect.conf` uses `default_server` to handle ACME challenges. Any nginx server block with a named `server_name` on port 80 will match before the default and bypass the ACME handler — those blocks must include their own `/.well-known/acme-challenge/` location.

## Connected Projects

- [[wiki/infrastructure/nginx-reverse-proxy]]
- [[wiki/infrastructure/gcp-external-monitoring]]
- [[wiki/concepts/dns-failover]]

## Sources

- [[wiki/sources/infrastructure-repo]] — DDNS setup, certbot commands, ACME gotcha
