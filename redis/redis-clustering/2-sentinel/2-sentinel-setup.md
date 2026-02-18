---
source_course: "redis-clustering"
source_lesson: "redis-clustering-sentinel-setup"
---

# Deploying Redis Sentinel

Let's set up a complete Sentinel deployment with 1 master, 2 replicas, and 3 Sentinels.

## Prerequisites

- Redis master at 192.168.1.10:6379
- Redis replica 1 at 192.168.1.11:6379
- Redis replica 2 at 192.168.1.12:6379
- 3 servers for Sentinels (can co-locate with Redis)

## Sentinel Configuration

Create `sentinel.conf` for each Sentinel:

```bash
# sentinel.conf
port 26379

# Announce IP (required in Docker/cloud environments)
sentinel announce-ip 192.168.1.20
sentinel announce-port 26379

# Monitor the master named "mymaster"
# Last parameter (2) is quorum
sentinel monitor mymaster 192.168.1.10 6379 2

# Master password (if required)
sentinel auth-pass mymaster your_password

# Time to detect failure
sentinel down-after-milliseconds mymaster 5000

# Failover timeout
sentinel failover-timeout mymaster 60000

# Parallel syncs during failover
sentinel parallel-syncs mymaster 1

# Sentinel password (recommended)
requirepass sentinel_password
```

## Starting Sentinels

```bash
# Start each Sentinel
redis-sentinel /path/to/sentinel.conf

# Or
redis-server /path/to/sentinel.conf --sentinel
```

## Checking Sentinel Status

```redis
# Connect to Sentinel
redis-cli -p 26379

# Get master info
SENTINEL MASTER mymaster

# Get replicas
SENTINEL REPLICAS mymaster

# Get Sentinel instances
SENTINEL SENTINELS mymaster

# Get current master address
SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
# Returns: 1) "192.168.1.10"
#          2) "6379"
```

## Sentinel Commands

```redis
# Force failover (for testing)
SENTINEL FAILOVER mymaster

# Check if master is alive
SENTINEL CKQUORUM mymaster

# Remove old/stale Sentinels
SENTINEL RESET mymaster

# Get configuration
SENTINEL CONFIG GET *
```

## Client Connection

Clients should connect to Sentinel to discover the master:

```python
import redis
from redis.sentinel import Sentinel

# Connect to Sentinel cluster
sentinel = Sentinel([
    ('192.168.1.20', 26379),
    ('192.168.1.21', 26379),
    ('192.168.1.22', 26379)
], socket_timeout=0.1)

# Get master connection
master = sentinel.master_for('mymaster', socket_timeout=0.1)
master.set('key', 'value')

# Get replica connection (for reads)
replica = sentinel.slave_for('mymaster', socket_timeout=0.1)
value = replica.get('key')
```

## Monitoring Failover

Subscribe to Sentinel events:

```redis
# Subscribe to all Sentinel events
redis-cli -p 26379 PSUBSCRIBE *

# Events include:
# +sdown       - SDOWN state
# -sdown       - Leaving SDOWN
# +odown       - ODOWN state  
# -odown       - Leaving ODOWN
# +switch-master - Master changed
# +failover-state-*  - Failover stages
```

## Testing Failover

```bash
# Kill the master
redis-cli -h 192.168.1.10 DEBUG SLEEP 30

# Or actually stop it
redis-cli -h 192.168.1.10 SHUTDOWN

# Watch Sentinel logs for failover
# After ~5 seconds (down-after-milliseconds), failover begins
```

📖 [Sentinel Deployment](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/#example-sentinel-deployments)

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*