---
source_course: "redis-clustering"
source_lesson: "redis-clustering-replication-psync"
---

# Partial Resynchronization and Replication IDs

## Introduction

When a replica temporarily loses connection to its master, Redis does not always need to perform a full resynchronization. The PSYNC mechanism allows replicas to resume replication from where they left off, saving significant time and bandwidth.

## Key Concepts

- **PSYNC**: The command replicas use to request partial or full resynchronization from the master
- **Replication ID**: A pseudo-random string identifying a specific dataset history; changes when a replica is promoted
- **Replication Offset**: A monotonically increasing integer tracking the byte position in the replication stream
- **Replication Backlog**: A fixed-size in-memory buffer on the master that stores recent replication stream data

## Real World Context

In production, brief network blips between master and replica are common. Without partial resync, every disconnection would trigger a full RDB transfer, which is expensive for large datasets. PSYNC makes Redis replication resilient to transient network issues.

## Deep Dive

### How PSYNC Works

When a replica reconnects, it sends:
```
PSYNC <replication-id> <offset>
```

The master checks:
1. Does the replication ID match my current or previous ID?
2. Is the requested offset still available in my backlog buffer?

If both conditions are met, the master sends only the missing data (partial resync). Otherwise, a full resync is triggered.

```
+---------------------------------------------------------------+
|  Replica reconnects:                                          |
|  PSYNC abc123... 50000                                        |
|                                                               |
|  Master checks backlog:                                       |
|  - Backlog has offsets 45000 - 60000                          |
|  - Offset 50000 is in range => PARTIAL RESYNC                |
|  - Sends only bytes 50000 - 60000                            |
+---------------------------------------------------------------+
```

### Replication Backlog Configuration

```bash
# Size of the replication backlog (default 1MB)
repl-backlog-size 256mb

# How long to retain backlog after all replicas disconnect
repl-backlog-ttl 3600
```

A larger backlog allows replicas to be disconnected for longer periods without requiring a full resync. Size it based on your write throughput and expected maximum disconnection time.

### Replication IDs Explained

Each Redis master has two replication IDs:

```redis
INFO replication
# master_replid:  abc123def456...
# master_replid2: 0000000000000000000000000000000000000000
# master_repl_offset: 50000
# second_repl_offset: -1
```

- **master_replid**: The current replication ID
- **master_replid2**: The previous replication ID (used after a failover)

When a replica is promoted to master, it:
1. Generates a new `master_replid`
2. Stores the old one in `master_replid2`
3. Records the offset at switch time in `second_repl_offset`

This allows other replicas that were connected to the old master to perform partial resync with the newly promoted master.

### Monitoring Resync Events

```bash
# Watch for full vs partial resyncs in Redis logs
# Full resync: "Full resync from master: abc123...:50000"
# Partial resync: "Successful partial resynchronization with master"

# Check replication backlog stats
redis-cli INFO replication | grep repl_backlog
# repl_backlog_active:1
# repl_backlog_size:268435456
# repl_backlog_first_byte_offset:1000
# repl_backlog_histlen:49000
```

### Redis 8 Improvements

Redis 8's dual-stream replication mechanism further reduces the chance of full resyncs by maintaining a 35% lower peak replication buffer. This means replicas can tolerate longer disconnections before the backlog overflows.

## Common Pitfalls

1. **Backlog too small for write throughput** -- If your write rate is 10 MB/s and the backlog is only 1 MB, any disconnection longer than 100ms triggers a full resync. Size the backlog to cover at least 60 seconds of writes.
2. **Forgetting replication ID changes on failover** -- After promoting a replica, other replicas need to recognize the new replication ID. Redis handles this automatically via `master_replid2`, but custom monitoring tools may need updates.

## Best Practices

1. **Size the backlog generously** -- Calculate your write throughput and multiply by your maximum acceptable disconnect duration. A 256 MB backlog covers most production scenarios.
2. **Monitor partial vs full resync ratio** -- If you see frequent full resyncs in your logs, increase the backlog size or investigate network stability.

## Summary

- PSYNC enables partial resynchronization, avoiding expensive full RDB transfers
- The replication backlog must be sized to cover expected disconnection windows
- Replication IDs track dataset history and enable partial resync after failover
- Redis 8 reduces peak buffer usage by 35%, further improving resync reliability
- Monitor `repl_backlog_histlen` to ensure your backlog is adequately sized

## Code Examples

**Configure and monitor the replication backlog for partial resynchronization**

```bash
# Configure replication backlog size
redis-cli CONFIG SET repl-backlog-size 256mb
redis-cli CONFIG SET repl-backlog-ttl 3600

# Check backlog status
redis-cli INFO replication | grep repl_backlog
```


## Resources

- [Redis Replication - Partial Resynchronization](https://redis.io/docs/latest/operate/oss_and_stack/management/replication/) — Official documentation covering PSYNC and replication backlog

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*