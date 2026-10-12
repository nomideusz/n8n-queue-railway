# Deploy and Host n8n (Queue Mode) on Railway

[![Deploy on Railway](https://railway.com/button.svg)](https://railway.com/deploy/n8n-queue?referralCode=sLjDXb&utm_medium=integration&utm_source=button&utm_campaign=n8n-queue)

[n8n](https://n8n.io/) is the workflow automation platform: connect 400+ services, build AI agents with built-in LangChain nodes, and automate anything from webhooks to scheduled jobs. This template runs it in **queue mode** — n8n's production architecture — instead of the single-process setup most templates ship.

## About Hosting n8n in Queue Mode

Five Railway services: the main instance (editor UI, queue producer), horizontally scalable workers (executions), a dedicated webhook processor (production webhooks keep flowing while you redeploy the editor), PostgreSQL 16 with **pgvector pre-enabled** (n8n's AI nodes get a vector store for free), and Redis as the Bull queue. The queue connection sets `QUEUE_BULL_REDIS_DUALSTACK=true` — BullMQ's ioredis defaults to IPv4 while Railway's private DNS is IPv6-only, which is the single most common way self-assembled n8n queue setups break here. Redis runs append-only on a volume so queued executions survive restarts. Scale throughput by increasing worker replicas — no other changes needed.

## Common Use Cases

- Production automation backbone: hundreds of executions per minute across scaled workers
- AI agent workflows with n8n's LangChain nodes, using pgvector for RAG without an extra database
- Webhook-heavy integrations (payment events, CRM syncs) that must not drop requests during deploys

## Dependencies for n8n Queue Mode Hosting

- All bundled: PostgreSQL 16 (pgvector), Redis 7, workers, and webhook processor are provisioned by the template

### Deployment Dependencies

- [n8n queue mode documentation](https://docs.n8n.io/hosting/scaling/queue-mode/)
- [Template source on GitHub](https://github.com/nomideusz/n8n-queue-railway)

### Implementation Details

**Your n8n editor is the `main` service's domain** — first visit creates the owner account. Production webhook URLs use the `webhook` service's domain automatically.

Service map:

| Service | Image | Role |
|---|---|---|
| main | `n8nio/n8n` | Editor UI + API + queue producer, public domain, volume at `/home/node/.n8n` |
| worker | `n8nio/n8n` (CMD `worker`) | Executes workflows; scale replicas for throughput |
| webhook | `n8nio/n8n` (CMD `webhook`) | Receives production webhooks, public domain |
| postgres | `pgvector/pgvector:pg16` | Workflows, credentials, executions + vector store, on a volume |
| redis | `redis:7-alpine` | Bull queue (append-only, on a volume) |

Variable wiring (lives in the template composer; documented here for maintenance):

- **Shared by main / worker / webhook**: `N8N_ENCRYPTION_KEY` (generated — must be identical on all three or credentials fail to decrypt), `DB_POSTGRESDB_HOST=${{postgres.RAILWAY_PRIVATE_DOMAIN}}`, `DB_POSTGRESDB_USER=postgres`, `DB_POSTGRESDB_PASSWORD=${{postgres.POSTGRES_PASSWORD}}`, `QUEUE_BULL_REDIS_HOST=${{redis.RAILWAY_PRIVATE_DOMAIN}}`, `QUEUE_BULL_REDIS_PASSWORD=${{redis.REDIS_PASSWORD}}`
- **main**: `N8N_HOST=${{RAILWAY_PUBLIC_DOMAIN}}`, `N8N_EDITOR_BASE_URL=https://${{RAILWAY_PUBLIC_DOMAIN}}`, `WEBHOOK_URL=https://${{webhook.RAILWAY_PUBLIC_DOMAIN}}/`, `PORT=5678` (explicit service variable — Railway injects `PORT=8080` over image ENV), `RAILWAY_RUN_UID=0` (the image runs as user `node`, Railway volumes mount root-owned; without it n8n crash-loops on `EACCES`)
- **webhook**: `WEBHOOK_URL=https://${{RAILWAY_PUBLIC_DOMAIN}}/`, `PORT=5678`
- **worker**: nothing extra (no domain, no volume — needs no PORT or UID override)
- **postgres**: `POSTGRES_PASSWORD` (generated); volume at `/var/lib/postgresql/data`
- **redis**: `REDIS_PASSWORD` (generated); volume at `/data`

Notes and limits:

- Workers default to concurrency 10; scale replicas (or set `N8N_CONCURRENCY_PRODUCTION_LIMIT`) as load grows.
- Manual editor executions are offloaded to workers too (`OFFLOAD_MANUAL_EXECUTIONS_TO_WORKERS=true` — no `N8N_` prefix on this one), so the main instance stays responsive.
- Worker health endpoints (`/healthz`, `/healthz/readiness`) are enabled for Railway healthchecks.
- All three n8n services must run the same image version; upgrade them together.

## Why Deploy n8n (Queue Mode) on Railway?

Railway is a singular platform to deploy your infrastructure stack. Railway will host your infrastructure so you don't have to deal with configuration, while allowing you to vertically and horizontally scale it.

By deploying n8n in queue mode on Railway, you are one step closer to supporting a complete full-stack application with minimal burden. Host your servers, databases, AI agents, and more on Railway.
