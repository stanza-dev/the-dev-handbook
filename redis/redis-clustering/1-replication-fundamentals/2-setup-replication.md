---
source_course: "redis-clustering"
source_lesson: "redis-clustering-setup-replication"
---

# Setting Up Replication

Let's configure a master-replica setup.

## Method 1: Configuration File

### Replica Configuration (redis.conf)

```bash
# On the replica server
# Point to the master
replicaof 192.168.1.10 6379

# If master requires authentication
masterauth your_master_password

# Make replica read-only (default, recommended)
replica-read-only yes
```

Start the replica:
```bash
redis-server /path/to/redis.conf
```

## Method 2: Runtime Command

```redis
# On the replica, connect and run:
REPLICAOF 192.168.1.10 6379

# To stop replication and become standalone master:
REPLICAOF NO ONE
```

## Complete Setup Example

### Master (192.168.1.10:6379)

```bash
# redis-master.conf
bind 0.0.0.0
port 6379
requirepass master_password

# Persistence
save 900 1
appendonly yes
```

### Replica 1 (192.168.1.11:6379)

```bash
# redis-replica1.conf
bind 0.0.0.0
port 6379
requirepass replica_password

# Replication
replicaof 192.168.1.10 6379
masterauth master_password
replica-read-only yes
```

### Replica 2 (192.168.1.12:6379)

```bash
# redis-replica2.conf
bind 0.0.0.0  
port 6379
requirepass replica_password

replicaof 192.168.1.10 6379
masterauth master_password
replica-read-only yes
```

## Verifying Replication

### On Master

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

### On Replica

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

## Testing Replication

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

## Chained Replication

Replicas can replicate from other replicas:

```
┌────────┐     ┌────────┐     ┌────────┐
│ Master │────▶│Replica1│────▶│Replica2│
└────────┘     └────────┘     └────────┘
```

```bash
# Replica 2 config
replicaof 192.168.1.11 6379  # Points to Replica 1
```

**Benefits**: Reduces load on master
**Drawback**: Additional replication lag

## Diskless Replication

For faster sync without writing RDB to disk:

```bash
# On master
repl-diskless-sync yes
repl-diskless-sync-delay 5  # Wait for more replicas before starting
```

📖 [REPLICAOF Command](https://redis.io/docs/latest/commands/replicaof/)

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*