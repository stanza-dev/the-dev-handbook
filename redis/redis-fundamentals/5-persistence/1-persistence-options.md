---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-persistence-options"
---

# Understanding Redis Persistence

## Introduction

Redis stores data in memory for speed, but without persistence configuration, all data is lost on restart. Understanding RDB snapshots and AOF logging lets you choose the right durability guarantee for each use case.

## Key Concepts

- **RDB (Redis Database)**: Creates point-in-time snapshots saved as a compact binary file. Fast recovery but may lose data between snapshots.
- **AOF (Append Only File)**: Logs every write operation. Slower recovery (replays commands) but minimal data loss.
- **BGSAVE**: Creates an RDB snapshot in a background fork without blocking Redis.
- **appendfsync**: Controls how often AOF data is flushed to disk — always, everysec, or no.

## Real World Context

A session cache might use no persistence at all (data is disposable). A shopping cart needs at least RDB (lose a few minutes is acceptable). A financial ledger needs AOF with fsync always (zero data loss). Choosing the right option balances performance against durability requirements.

## Deep Dive

Redis stores data in memory for speed, but offers persistence options to survive restarts. Understanding these options is crucial for choosing the right durability guarantees.

## Why Persistence Matters

```
┌─────────────────────────────────────────────────────────┐
│                    Without Persistence                    │
│  Server restart = ALL DATA LOST                          │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    With Persistence                       │
│  Server restart = Data recovered from disk               │
└─────────────────────────────────────────────────────────┘
```

## Persistence Options

| Option | Description | Durability | Performance |
|--------|-------------|------------|-------------|
| None | No persistence | Data lost on restart | Fastest |
| RDB | Point-in-time snapshots | May lose recent data | Fast |
| AOF | Log every write | Minimal data loss | Slower |
| RDB + AOF | Both methods | Best durability | Balanced |

## RDB (Redis Database)

RDB creates point-in-time snapshots of your dataset.

### How RDB Works

```
┌──────────────────────────────────────────┐
│  Time 0:00    │    Time 5:00 (save)      │
│  SET a 1      │    [snapshot.rdb]        │
│  SET b 2      │    contains a=1, b=2     │
└──────────────────────────────────────────┘
         ↓
  If crash at 5:01, data from 5:00 is safe
  But commands between 5:00-5:01 are lost
```

### RDB Configuration

```bash
# In redis.conf
# Save after 900 seconds if at least 1 key changed
save 900 1

# Save after 300 seconds if at least 10 keys changed
save 300 10

# Save after 60 seconds if at least 10000 keys changed
save 60 10000

# RDB filename
dbfilename dump.rdb

# Directory for RDB file
dir /var/lib/redis
```

### Manual RDB Commands

```redis
# Create snapshot (blocks Redis)
SAVE

# Create snapshot in background (recommended)
BGSAVE

# Check last save time
LASTSAVE

# Get RDB file location
CONFIG GET dir
CONFIG GET dbfilename
```

### RDB Pros and Cons

**Pros:**
- Compact single-file backup
- Perfect for backups and disaster recovery
- Faster restarts (loads RDB directly)
- Minimal performance impact during normal operation

**Cons:**
- May lose data between snapshots
- Fork operation can be slow on large datasets
- Not suitable for zero data loss requirements

## AOF (Append Only File)

AOF logs every write operation, allowing complete data reconstruction.

### How AOF Works

```
┌─────────────────────────────────────────┐
│  Commands      │  appendonly.aof        │
│  SET a 1       │  *3\r\n$3\r\nSET...    │
│  SET b 2       │  *3\r\n$3\r\nSET...    │
│  INCR a        │  *2\r\n$4\r\nINCR...   │
└─────────────────────────────────────────┘
         ↓
  On restart, replay all commands = exact state
```

### AOF Configuration

```bash
# Enable AOF
appendonly yes

# AOF filename
appendfilename "appendonly.aof"

# Sync policy (when to write to disk)
appendfsync always    # Every write (safest, slowest)
appendfsync everysec  # Every second (good balance)
appendfsync no        # Let OS decide (fastest, risky)
```

### AOF Pros and Cons

**Pros:**
- Much more durable (can lose only 1 second of data)
- Append-only is corruption-resistant
- Human-readable format
- Automatic rewriting to keep file size reasonable

**Cons:**
- Larger files than RDB
- Slower restarts (must replay all commands)
- Slightly slower for writes (depending on fsync policy)

📖 [Redis Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/)

## Common Pitfalls

1. **Assuming Redis persists by default** — Some installations disable persistence. Always verify with INFO persistence that your chosen method is active.
2. **Using SAVE in production** — SAVE blocks Redis for the entire duration of the snapshot. Always use BGSAVE for non-blocking snapshots.

## Best Practices

1. **Use RDB + AOF together** — RDB provides fast restarts and compact backups; AOF fills the durability gap between snapshots.
2. **Set appendfsync to everysec** — This is the recommended default, losing at most one second of data while maintaining good performance.

## Summary

- RDB creates periodic snapshots; AOF logs every write.
- BGSAVE creates non-blocking RDB snapshots in a fork.
- AOF with everysec loses at most 1 second of data on crash.
- RDB + AOF together provides the best of both worlds.
- Redis loads from AOF on startup when both are enabled (AOF is more complete).

## Code Examples

**The two persistence mechanisms — BGSAVE creates RDB snapshots, AOF logs every write for maximum durability**

```bash
# RDB: Create a background snapshot
BGSAVE

# AOF: Enable in redis.conf
# appendonly yes
# appendfsync everysec

# Check persistence status
INFO persistence
# rdb_last_bgsave_status:ok
# aof_enabled:1
```


## Resources

- [Redis Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) — Complete guide to Redis persistence options

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*