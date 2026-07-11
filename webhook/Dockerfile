FROM n8nio/n8n:2.29.10

# Dedicated webhook processor: production webhook traffic keeps flowing
# while the main instance restarts or redeploys.
ENV EXECUTIONS_MODE=queue \
    DB_TYPE=postgresdb \
    DB_POSTGRESDB_PORT=5432 \
    DB_POSTGRESDB_DATABASE=n8n \
    DB_POSTGRESDB_USER=postgres \
    QUEUE_BULL_REDIS_PORT=6379 \
    QUEUE_BULL_REDIS_DUALSTACK=true \
    N8N_PROTOCOL=https \
    N8N_PORT=5678 \
    N8N_PROXY_HOPS=1

CMD ["webhook"]
