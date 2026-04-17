---
title: "GCP Cloud Run"
type: concept
tags: [concept, gcp, cloud-run, serverless, containers]
created: 2026-04-17
updated: 2026-04-17
source_count: 0
---

# GCP Cloud Run

Cloud Run is Google Cloud's managed serverless container platform. You deploy a Docker image; GCP handles scaling, SSL, load balancing, and infrastructure. Instances scale to zero when idle (no traffic = no cost) and spin up on demand. Billed per request and per CPU/memory second.

## How It Is Used Here

Two services run on Cloud Run in GCP project `cameron-tora` (`us-central1`):

| Service | Purpose | Why Cloud Run |
|---------|---------|---------------|
| [[wiki/infrastructure/status-dashboard\|Status Dashboard]] | Home server health monitoring | Must be reachable when home server is down — can't self-host a monitor |
| [[wiki/infrastructure/gcp-external-monitoring\|GCP External Monitor]] | Alerting + DNS failover automation | Same reason; also runs scheduled health checks |

Both services exist specifically because they monitor the home server — hosting them on the home server would make them useless during an outage.

## Key Properties / Mechanics

### Deployment Model

```bash
# Build image via Cloud Build, deploy to Cloud Run
gcloud run deploy <service-name> \
  --image gcr.io/<project>/<image> \
  --region us-central1 \
  --allow-unauthenticated
```

The status dashboard uses a `deploy.sh` script that checks for required secrets, builds via Cloud Build, and deploys. Triggered manually from Cloud Shell or locally with `gcloud`.

### Scaling to Zero

Min instances = 0 by default. The first request after idle incurs a **cold start** (~1–3 seconds for Python/Node services). For the status dashboard this is acceptable — it's queried by humans and by Cloud Scheduler (which is not latency-sensitive).

The status dashboard is configured with `min_instances: 0` and `max_instances: 2`.

### Cloud Scheduler Integration

Cloud Scheduler triggers Cloud Run services on a cron schedule via HTTP POST:

```
*/5 * * * *  →  POST status-dashboard/api/check
*/5 * * * *  →  POST gcp-monitor/check
```

This drives the 5-minute health check cycle. Cloud Scheduler retries on failure.

### Secret Manager

Environment variables containing secrets (API keys, webhook URLs) are stored in GCP Secret Manager and injected at deploy time — never baked into the Docker image or committed to git.

| Secret | Used by |
|--------|---------|
| `health-api-key` | Status dashboard → health.camerontora.ca |
| `discord-webhook-url` | Both services → Discord alerts |
| `godaddy-api-key` / `godaddy-api-secret` | Status dashboard → DNS failover |
| `admin-api-key` | Status dashboard admin panel |

### Domain Mapping

`status.camerontora.ca` is a **CNAME to `ghs.googlehosted.com`** — SSL is managed by Google, not Let's Encrypt. This means:
- `status.camerontora.ca` must **not** be added to the `camerontora-services` Let's Encrypt cert
- It also doubles as the DNS failover landing page — when `@` and `plex` flip to GCP IPs during an outage, visitors see the status dashboard

### Firestore (Companion Service)

The status dashboard writes 5-minute snapshots to Cloud Firestore (Native mode, `us-central1`) for historical uptime tracking. 7-day retention, free tier handles the volume.

### Cost Profile

At low traffic (one Cloud Scheduler trigger per 5 minutes, occasional human visits): effectively free under GCP's free tier limits for Cloud Run (2 million requests/month free) and Firestore (50K reads/day free).

## Relationships to Other Concepts

- [[wiki/concepts/docker-compose-networking]] — contrast: Cloud Run abstracts all networking; no compose files
- [[wiki/concepts/webhook-patterns]] — Cloud Run services receive webhooks from Cloud Scheduler

## External References

- Cloud Run docs: https://cloud.google.com/run/docs
- Cloud Scheduler: https://cloud.google.com/scheduler/docs
