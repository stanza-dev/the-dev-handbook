---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-persistence-monitoring"
---

# Monitoring Persistence Health

## Introduction

Configuring persistence is only half the battle — you need to monitor it continuously to catch problems before they cause data loss. Redis provides detailed metrics through the INFO command that tell you exactly how your persistence layer is performing.

## Key Concepts

- **INFO persistence**: A Redis command that returns all persistence-related metrics in one call.
- **rdb_changes_since_last_save**: Number of write operations since the last successful RDB snapshot — a high number means more data at risk.
- **aof_current_size**: The current size of the AOF file on disk.
- **rdb_last_bgsave_status**: Whether the last background save succeeded or failed.

## Real World Context

In production, a silently failing BGSAVE can go unnoticed for days. If Redis crashes during that window, you lose all data since the last successful save. Monitoring persistence metrics and setting up alerts is essential for any Redis deployment that stores important data.

## Deep Dive

### INFO Persistence Output

```redis
INFO persistence

# RDB metrics:
# rdb_last_save_time:1704067200
# rdb_changes_since_last_save:150
# rdb_last_bgsave_status:ok
# rdb_last_bgsave_time_sec:2
# rdb_current_bgsave_time_sec:-1

# AOF metrics:
# aof_enabled:1
# aof_last_rewrite_status:ok
# aof_last_write_status:ok
# aof_current_size:1048576
# aof_buffer_length:0
```

### Key Metrics to Monitor

| Metric | Warning Sign | Action |
|--------|-------------|--------|
| rdb_last_bgsave_status | err | Check disk space, permissions |
| rdb_changes_since_last_save | Continuously growing | BGSAVE may be failing |
| aof_last_write_status | err | Disk may be full |
| aof_current_size | Very large | Trigger BGREWRITEAOF |
| aof_buffer_length | Non-zero for long periods | Disk I/O bottleneck |

### Checking Persistence from CLI

```redis
# Quick health check
INFO persistence

# Check last save timestamp
LASTSAVE
# Returns: Unix timestamp of last successful RDB save

# Check server stats including persistence
INFO stats
# Look for: total_commands_processed, instantaneous_ops_per_sec
```

### Automated Health Script

```bash
#!/bin/bash
# check-redis-health.sh

RDB_STATUS=$(redis-cli INFO persistence | grep rdb_last_bgsave_status | cut -d: -f2 | tr -d '\r')
AOF_STATUS=$(redis-cli INFO persistence | grep aof_last_write_status | cut -d: -f2 | tr -d '\r')
CHANGES=$(redis-cli INFO persistence | grep rdb_changes_since_last_save | cut -d: -f2 | tr -d '\r')

if [ "$RDB_STATUS" != "ok" ]; then
    echo "ALERT: RDB save failing!"
fi

if [ "$AOF_STATUS" != "ok" ]; then
    echo "ALERT: AOF write failing!"
fi

if [ "$CHANGES" -gt 10000 ]; then
    echo "WARNING: $CHANGES changes since last save"
fi
```

### Troubleshooting Common Issues

**BGSAVE failing:**
```redis
# Check error in logs
# Common causes:
# 1. Not enough memory for fork (need ~2x during save)
# 2. Disk full
# 3. Permission denied on dump.rdb

# Quick fix: manual save (blocks Redis)
SAVE

# Better fix: ensure sufficient memory
INFO memory
# Check used_memory vs available system RAM
```

**AOF file growing too large:**
```redis
# Trigger manual rewrite
BGREWRITEAOF

# Check rewrite progress
INFO persistence
# aof_rewrite_in_progress:1
```

## Common Pitfalls

1. **Not monitoring rdb_last_bgsave_status** — BGSAVE failures are silent. Without monitoring, you may think your data is safe when it is not.
2. **Ignoring AOF file size** — A constantly growing AOF slows restarts and wastes disk. Ensure auto-rewrite thresholds are configured.
3. **Insufficient memory for fork** — BGSAVE forks the Redis process, temporarily doubling memory usage. If the system lacks headroom, the fork fails.

## Best Practices

1. **Set up alerts on persistence metrics** — Monitor rdb_last_bgsave_status, aof_last_write_status, and rdb_changes_since_last_save with your monitoring stack.
2. **Test recovery regularly** — Periodically restore from your RDB/AOF backups to verify they actually work.
3. **Keep disk usage below 80%** — Leave room for AOF rewrites, RDB snapshots, and temporary files.

## Summary

- Use INFO persistence to check RDB and AOF health metrics.
- Monitor rdb_last_bgsave_status and aof_last_write_status for failures.
- Watch rdb_changes_since_last_save to detect stalled backups.
- Automate health checks and set up alerts for production systems.
- Test disaster recovery procedures before you actually need them.

## Code Examples

**One-liner to check the three most critical persistence metrics — bgsave status, AOF write status, and unsaved changes**

```bash
# Quick persistence health check
redis-cli INFO persistence | grep -E 'rdb_last_bgsave_status|aof_last_write_status|rdb_changes_since_last_save'
# rdb_last_bgsave_status:ok
# aof_last_write_status:ok
# rdb_changes_since_last_save:42
```


## Resources

- [Redis Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) — Official Redis persistence documentation

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*