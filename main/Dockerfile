FROM n8nio/n8n:2.29.10

# Main instance: editor UI + API + queue producer. Executions run on the
# worker service; webhook traffic lands on the webhook service.
# QUEUE_BULL_REDIS_DUALSTACK is required on Railway: BullMQ's ioredis
# defaults to IPv4 and Railway private DNS is IPv6-only.
ENV EXECUTIONS_MODE=queue \
    N8N_OFFLOAD_MANUAL_EXECUTIONS_TO_WORKERS=true \
    DB_TYPE=postgresdb \
    DB_POSTGRESDB_PORT=5432 \
    DB_POSTGRESDB_DATABASE=n8n \
    QUEUE_BULL_REDIS_PORT=6379 \
    QUEUE_BULL_REDIS_DUALSTACK=true \
    N8N_PROTOCOL=https \
    N8N_PORT=5678 \
    N8N_PROXY_HOPS=1 \
    N8N_RUNNERS_ENABLED=true
