---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-persistence-hybrid"
---

# Hybrid Persistence and Backups

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

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*