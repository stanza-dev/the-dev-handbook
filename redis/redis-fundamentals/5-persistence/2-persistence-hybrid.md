---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-persistence-hybrid"
---

# Hybrid Persistence and Backups

## Introduction

For production systems that need both fast recovery and strong durability, combining RDB and AOF provides the best of both worlds. This lesson covers how to configure hybrid persistence, manage AOF rewriting, and implement backup strategies.

## Key Concepts

- **Hybrid persistence**: Running RDB and AOF simultaneously — RDB for fast backup/restore, AOF for minimal data loss.
- **AOF rewriting**: BGREWRITEAOF compacts the AOF by replacing the command log with the current dataset state.
- **Startup priority**: When both are enabled, Redis loads from AOF on restart because it is more complete.
- **Copy-on-write**: Redis uses OS copy-on-write during BGSAVE, making it safe to copy RDB files while Redis is running.

## Real World Context

Production Redis deployments at companies like GitHub and Instagram use hybrid persistence. The RDB file provides a compact, easy-to-transfer backup for disaster recovery, while AOF ensures that the gap between the last RDB and a crash loses at most one second of data.

## Deep Dive

For production systems, combining RDB and AOF provides the best of both worlds.

## RDB + AOF (Recommended)

```bash
# Enable both in redis.conf
save 900 1
save 300 10
appendonly yes
appendfsync everysec
```

### How Redis Chooses on Restart

```
┌─────────────────────────────────────────────┐
│  On startup, Redis checks:                   │
│  1. Is AOF enabled? → Load from AOF          │
│  2. No AOF? → Load from RDB                  │
│                                              │
│  AOF is preferred because it's more complete │
└─────────────────────────────────────────────┘
```

## AOF Rewriting

AOF files grow over time. Rewriting compacts them:

```
Before rewrite:           After rewrite:
┌──────────────────┐      ┌──────────────────┐
│ SET x 1          │      │ SET x 100        │
│ SET x 2          │  →   │ SET y "final"    │
│ SET x 100        │      └──────────────────┘
│ SET y "hello"    │      (Just final state)
│ SET y "final"    │
└──────────────────┘
```

```redis
# Trigger rewrite manually
BGREWRITEAOF

# Auto-rewrite configuration
auto-aof-rewrite-percentage 100    # Rewrite when 100% bigger than last rewrite
auto-aof-rewrite-min-size 64mb     # Don't rewrite if smaller than 64MB
```

## Backup Strategies

### Automated RDB Backups

```bash
#!/bin/bash
# backup-redis.sh

REDIS_DIR="/var/lib/redis"
BACKUP_DIR="/backups/redis"
DATE=$(date +%Y%m%d_%H%M%S)

# Trigger a background save
redis-cli BGSAVE

# Wait for save to complete
while [ $(redis-cli LASTSAVE) == $(redis-cli LASTSAVE) ]; do
    sleep 1
done

# Copy the RDB file
cp $REDIS_DIR/dump.rdb $BACKUP_DIR/dump_$DATE.rdb

# Keep only last 7 days
find $BACKUP_DIR -name "dump_*.rdb" -mtime +7 -delete
```

### Copy-on-Write Safety

RDB is safe to copy even while Redis is running:

```bash
# Safe: Redis uses copy-on-write during BGSAVE
cp /var/lib/redis/dump.rdb /backups/

# Also safe: Copy while BGSAVE is running
```

## Checking Persistence Status

```redis
# Get persistence info
INFO persistence

# Key fields:
# rdb_last_save_time:1704067200
# rdb_changes_since_last_save:150
# aof_enabled:1
# aof_last_rewrite_time_sec:10
# aof_current_size:1048576
```

## Disaster Recovery

### Recovering from RDB

```bash
# 1. Stop Redis
sudo systemctl stop redis

# 2. Replace dump.rdb with backup
cp /backups/dump_20240115.rdb /var/lib/redis/dump.rdb

# 3. Set correct ownership
chown redis:redis /var/lib/redis/dump.rdb

# 4. Start Redis
sudo systemctl start redis
```

### Recovering from AOF

```bash
# If AOF is corrupted, fix it first
redis-check-aof --fix /var/lib/redis/appendonly.aof

# Then start Redis normally
```

## Choosing the Right Strategy

| Use Case | Recommendation |
|----------|----------------|
| Cache only | No persistence |
| Session store | RDB only |
| Primary database | RDB + AOF |
| Financial data | AOF with fsync always |

📖 [Persistence FAQ](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)

## Common Pitfalls

1. **Not monitoring AOF file size** — Without auto-rewrite, AOF files grow continuously and slow down restarts. Ensure auto-aof-rewrite-percentage and auto-aof-rewrite-min-size are configured.
2. **Assuming backups are valid** — An RDB file could be corrupted. Periodically test recovery by restoring backups to a test instance.

## Best Practices

1. **Automate RDB backups with BGSAVE + cp** — Schedule regular BGSAVE and copy the dump.rdb to external storage for disaster recovery.
2. **Keep auto-rewrite thresholds reasonable** — 100% growth trigger and 64MB minimum size prevents excessive rewrites on small datasets.

## Summary

- Enable both RDB and AOF for production persistence.
- Redis loads from AOF on startup when both are enabled.
- BGREWRITEAOF compacts AOF files to prevent unbounded growth.
- RDB files are safe to copy while Redis is running (copy-on-write).
- Test backup recovery regularly — untested backups are not backups.

## Code Examples

**Configuring RDB + AOF hybrid persistence — the recommended production setup for maximum durability**

```bash
# Enable hybrid persistence in redis.conf
save 900 1
save 300 10
appendonly yes
appendfsync everysec

# Check persistence status
redis-cli INFO persistence | grep -E 'rdb_last_bgsave|aof_enabled'
```


## Resources

- [Redis Persistence FAQ](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) — Frequently asked questions about Redis persistence

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*