---
source_course: "redis-clustering"
source_lesson: "redis-clustering-sentinel-setup"
---

# Deploying Redis Sentinel

## Introduction

Deploying a Sentinel cluster requires careful planning of network topology, configuration files, and client integration. This lesson walks through a complete 3-Sentinel deployment with practical commands for every step.

## Key Concepts

- **sentinel.conf**: The configuration file for each Sentinel instance, separate from redis.conf
- **announce-ip**: Required in cloud and Docker environments where the internal IP differs from the reachable IP
- **parallel-syncs**: Controls how many replicas can sync with the new master simultaneously during failover
- **SENTINEL FAILOVER**: Manual command to trigger failover for testing or planned maintenance

## Real World Context

Setting up Sentinel is one of the first operational tasks when moving Redis to production. Getting the configuration right from the start prevents failover surprises under real outage conditions.

## Deep Dive

### Prerequisites

- Redis master at 192.168.1.10:6379
- Redis replica 1 at 192.168.1.11:6379
- Redis replica 2 at 192.168.1.12:6379
- 3 servers for Sentinels (can co-locate with Redis)

### Sentinel Configuration

Create `sentinel.conf` for each Sentinel:

```bash
port 26379
sentinel announce-ip 192.168.1.20
sentinel announce-port 26379
sentinel monitor mymaster 192.168.1.10 6379 2
sentinel auth-pass mymaster your_password
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
requirepass sentinel_password
```

### Starting Sentinels

```bash
redis-sentinel /path/to/sentinel.conf
# Or
redis-server /path/to/sentinel.conf --sentinel
```

### Checking Sentinel Status

```redis
redis-cli -p 26379
SENTINEL MASTER mymaster
SENTINEL REPLICAS mymaster
SENTINEL SENTINELS mymaster
SENTINEL GET-MASTER-ADDR-BY-NAME mymaster
```

### Sentinel Commands

```redis
SENTINEL FAILOVER mymaster
SENTINEL CKQUORUM mymaster
SENTINEL RESET mymaster
SENTINEL CONFIG GET *
```

### Client Connection

```python
import redis
from redis.sentinel import Sentinel

sentinel = Sentinel([
    ('192.168.1.20', 26379),
    ('192.168.1.21', 26379),
    ('192.168.1.22', 26379)
], socket_timeout=0.1)

master = sentinel.master_for('mymaster', socket_timeout=0.1)
master.set('key', 'value')

replica = sentinel.slave_for('mymaster', socket_timeout=0.1)
value = replica.get('key')
```

### Monitoring Failover

```redis
redis-cli -p 26379 PSUBSCRIBE *
# Events: +sdown, -sdown, +odown, -odown, +switch-master, +failover-state-*
```

### Testing Failover

```bash
redis-cli -h 192.168.1.10 DEBUG SLEEP 30
# Or
redis-cli -h 192.168.1.10 SHUTDOWN
# After ~5 seconds (down-after-milliseconds), failover begins
```

## Common Pitfalls

1. **Missing announce-ip in Docker/cloud** -- Sentinel will advertise internal container IPs that other nodes cannot reach. Always set `sentinel announce-ip` and `sentinel announce-port` in containerized environments.
2. **Not testing failover before production** -- Many teams discover configuration issues only during real outages. Always test with `SENTINEL FAILOVER` or `DEBUG SLEEP` before going live.

## Best Practices

1. **Use parallel-syncs 1** -- During failover, only one replica syncs with the new master at a time, keeping other replicas available for reads.
2. **Subscribe to Sentinel events** -- Use `PSUBSCRIBE *` in your monitoring system to track failover events and alert on unexpected state changes.

## Summary

- Deploy at least 3 Sentinels with consistent configuration files
- Set announce-ip/port in cloud and Docker environments
- Clients should use Sentinel for master discovery, not hardcoded IPs
- Test failover before production with SENTINEL FAILOVER or DEBUG SLEEP
- Monitor Sentinel Pub/Sub events for visibility into failover activity

## Code Examples

**Connect to Redis through Sentinel for automatic master discovery**

```python
from redis.sentinel import Sentinel

sentinel = Sentinel([
    ('192.168.1.20', 26379),
    ('192.168.1.21', 26379),
    ('192.168.1.22', 26379)
], socket_timeout=0.1)

master = sentinel.master_for('mymaster', socket_timeout=0.1)
master.set('key', 'value')

replica = sentinel.slave_for('mymaster', socket_timeout=0.1)
value = replica.get('key')
```


## Resources

- [Sentinel Deployment](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/) — Sentinel deployment examples and best practices

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*