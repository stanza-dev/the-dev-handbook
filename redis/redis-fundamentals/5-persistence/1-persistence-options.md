---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-persistence-options"
---

# Understanding Redis Persistence

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

## Resources

- [Redis Persistence](https://redis.io/docs/latest/operate/oss_and_stack/management/persistence/) — Complete guide to Redis persistence options

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*