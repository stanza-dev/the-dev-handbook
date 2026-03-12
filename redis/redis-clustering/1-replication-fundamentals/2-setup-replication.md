---
source_course: "redis-clustering"
source_lesson: "redis-clustering-setup-replication"
---

# Setting Up Replication

## Introduction

Configuring Redis replication involves pointing a replica server at a master. Redis supports both configuration-file and runtime approaches, giving you flexibility in how you manage your topology.

## Key Concepts

- **REPLICAOF**: The modern command to configure a server as a replica (replaces deprecated SLAVEOF)
- **Chained Replication**: Replicas can replicate from other replicas to reduce master load
- **Diskless Replication**: Sync without writing RDB to disk for faster initial synchronization
- **masterauth**: Password the replica uses to authenticate with the master

## Real World Context

Every production Redis setup begins with replication. Whether you are running a simple primary-standby pair or a large fan-out topology, understanding the configuration options ensures reliable data copies across your infrastructure.

## Deep Dive

### Method 1: Configuration File

```bash
# On the replica server (redis.conf)
replicaof 192.168.1.10 6379
masterauth your_master_password
replica-read-only yes
```

Start the replica:
```bash
redis-server /path/to/redis.conf
```

### Method 2: Runtime Command

```redis
REPLICAOF 192.168.1.10 6379

# To stop replication and become standalone master:
REPLICAOF NO ONE
```

### Complete Setup Example

**Master (192.168.1.10:6379)**

```bash
# redis-master.conf
bind 0.0.0.0
port 6379
requirepass master_password
save 900 1
appendonly yes
```

**Replica 1 (192.168.1.11:6379)**

```bash
# redis-replica1.conf
bind 0.0.0.0
port 6379
requirepass replica_password
replicaof 192.168.1.10 6379
masterauth master_password
replica-read-only yes
```

**Replica 2 (192.168.1.12:6379)**

```bash
# redis-replica2.conf
bind 0.0.0.0
port 6379
requirepass replica_password
replicaof 192.168.1.10 6379
masterauth master_password
replica-read-only yes
```

### Verifying Replication

**On Master:**

```redis
127.0.0.1:6379> INFO replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.1.11,port=6379,state=online,offset=1234,lag=0
slave1:ip=192.168.1.12,port=6379,state=online,offset=1234,lag=0
master_replid:abc123...
master_repl_offset:1234
```

**On Replica:**

```redis
127.0.0.1:6379> INFO replication
# Replication
role:slave
master_host:192.168.1.10
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_read_only:1
```

### Testing Replication

```bash
# On master
redis-cli -h 192.168.1.10 SET mykey "Hello"

# On replica (should see the value)
redis-cli -h 192.168.1.11 GET mykey
# "Hello"

# Writing to replica should fail
redis-cli -h 192.168.1.11 SET mykey "World"
# (error) READONLY You can't write against a read only replica.
```

### Chained Replication

Replicas can replicate from other replicas:

```
Master --> Replica1 --> Replica2
```

```bash
# Replica 2 config
replicaof 192.168.1.11 6379  # Points to Replica 1
```

**Benefits**: Reduces load on master
**Drawback**: Additional replication lag

### Diskless Replication

For faster sync without writing RDB to disk:

```bash
# On master
repl-diskless-sync yes
repl-diskless-sync-delay 5  # Wait for more replicas before starting
```

## Common Pitfalls

1. **Forgetting masterauth** -- If the master requires a password, the replica will fail to connect silently. Always configure `masterauth` when `requirepass` is set on the master.
2. **Running replicas without persistence** -- If both master and replica have persistence disabled and the master restarts empty, the replica will also wipe its data during resync.

## Best Practices

1. **Use configuration files for production** -- Runtime REPLICAOF commands are lost on restart. Always persist replication settings in redis.conf.
2. **Enable diskless replication for large datasets** -- Diskless sync avoids the disk I/O bottleneck when datasets exceed several GB.

## Summary

- Use `REPLICAOF` (config file or runtime) to set up replication
- Verify with `INFO replication` on both master and replica
- Chained replication reduces master load at the cost of extra lag
- Diskless replication speeds up initial sync for large datasets
- Always test write rejection on replicas to confirm read-only mode

## Code Examples

**Two methods for configuring a Redis server as a replica**

```bash
# Configure replica via redis.conf
replicaof 192.168.1.10 6379
masterauth your_master_password
replica-read-only yes

# Or at runtime
redis-cli REPLICAOF 192.168.1.10 6379
```


## Resources

- [REPLICAOF Command](https://redis.io/docs/latest/commands/replicaof/) — Official REPLICAOF command reference

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*