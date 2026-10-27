# upgrade: npm n8n
FROM n8nio/n8n:2.45.4

# Queue consumer — scale replicas horizontally for throughput. Stateless:
# shares state via Postgres + the N8N_ENCRYPTION_KEY env (must match main).
# Health endpoints /healthz + /healthz/readiness on :5678 (binds ::).
ENV EXECUTIONS_MODE=queue \
    DB_TYPE=postgresdb \
    DB_POSTGRESDB_PORT=5432 \
    DB_POSTGRESDB_DATABASE=n8n \
    DB_POSTGRESDB_USER=postgres \
    QUEUE_BULL_REDIS_PORT=6379 \
    QUEUE_BULL_REDIS_DUALSTACK=true \
    QUEUE_HEALTH_CHECK_ACTIVE=true \
    N8N_RUNNERS_ENABLED=true

CMD ["worker"]
