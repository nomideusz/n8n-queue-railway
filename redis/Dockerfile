FROM redis:7-alpine

# No config file → redis binds all interfaces including IPv6 (required on
# Railway's private network). requirepass satisfies protected-mode.
# appendonly persists the Bull queue across restarts (queued executions
# survive a redis redeploy); pair with a small volume at /data.
CMD ["sh", "-c", "redis-server --appendonly yes --requirepass \"$REDIS_PASSWORD\""]
