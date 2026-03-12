---
source_course: "redis-clustering"
source_lesson: "redis-clustering-metrics"
---

# Key Metrics and Health Indicators

## Introduction

Monitor the right metrics to maintain a healthy Redis cluster. Effective monitoring catches problems early, before they impact users.

## Key Concepts

- **used_memory vs used_memory_rss**: Redis-tracked memory vs OS-reported memory; their ratio reveals fragmentation
- **instantaneous_ops_per_sec**: Real-time throughput counter from INFO stats
- **keyspace_hits/misses**: The raw counters used to calculate cache hit rate
- **cluster_state**: The ok/fail flag from CLUSTER INFO that indicates overall cluster health

## Real World Context

Most Redis incidents have precursors visible in metrics hours or days before the outage. Memory creeping toward maxmemory, replication lag increasing, or hit rate dropping all signal problems that can be addressed proactively.

## Deep Dive

### Memory Metrics

```redis
INFO memory
# used_memory: Total allocated memory
# used_memory_rss: Memory from OS perspective
# mem_fragmentation_ratio: RSS / used_memory
# maxmemory: Configured limit
```

| Metric | Alert Threshold | Action |
|--------|-----------------|--------|
| used_memory/maxmemory | > 80% | Warn |
| used_memory/maxmemory | > 95% | Critical |
| mem_fragmentation_ratio | > 1.5 | Investigate |
| mem_fragmentation_ratio | < 1.0 | Swapping! |

### Performance Metrics

```redis
INFO stats
# instantaneous_ops_per_sec: Current throughput
# keyspace_hits / keyspace_misses: Cache hit ratio
# total_connections_received: Connection rate
# rejected_connections: Connection refusals
```

### Replication Metrics

```redis
INFO replication
# Master perspective:
# connected_slaves: Number of replicas
# slave0: ip=...,port=...,state=online,offset=...,lag=0
#
# Replica perspective:
# master_link_status: up/down
# master_last_io_seconds_ago: Time since last communication
# slave_repl_offset: Replication position
```

```python
def check_replication_health(r):
    info = r.info('replication')
    if info['role'] == 'master':
        for i in range(info.get('connected_slaves', 0)):
            slave_info = info.get(f'slave{i}', '')
            lag = int(slave_info.split('lag=')[1].split(',')[0])
            if lag > 10:
                alert(f'Replica {i} lag: {lag} seconds')
    elif info['role'] == 'slave':
        if info.get('master_link_status') != 'up':
            alert('Replica disconnected from master!')
```

### Cluster-Specific Metrics

```redis
CLUSTER INFO
# cluster_state: ok/fail
# cluster_slots_assigned: Should be 16384
# cluster_slots_ok: Slots with active masters
# cluster_known_nodes: Total nodes in cluster
```

### Monitoring Tools

**Redis CLI Monitoring:**
```bash
redis-cli MONITOR       # Real-time (high overhead!)
redis-cli SLOWLOG GET 10  # Better: sample slow queries
```

**Prometheus Integration:**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

## Common Pitfalls

1. **Monitoring only memory** -- Memory is important but not sufficient. Replication lag, slow queries, and connection counts are equally critical indicators.
2. **Not setting up alerting** -- Dashboards are useful but only if someone is watching. Configure automated alerts for critical thresholds.

## Best Practices

1. **Use Prometheus + Grafana with redis-exporter** -- This stack provides comprehensive dashboards, historical data, and flexible alerting out of the box.
2. **Monitor the cache hit rate** -- A hit rate below 90% often indicates cache keys are expiring too quickly or the cache is not being used effectively.

## Summary

- Monitor memory, performance, replication, and cluster metrics via INFO
- Set warning alerts at 80% memory and critical at 95%
- Watch mem_fragmentation_ratio: below 1.0 means swapping
- Use Prometheus + Grafana for production-grade monitoring
- Cache hit rate below 90% signals ineffective caching strategy

## Code Examples

**Python script to check Redis memory usage and cache hit rate**

```python
import redis

r = redis.Redis()
info = r.info('memory')
used = info['used_memory']
max_mem = info.get('maxmemory', 0)
if max_mem > 0:
    usage = used / max_mem * 100
    print(f'Memory usage: {usage:.1f}%')

stats = r.info('stats')
hits = stats['keyspace_hits']
misses = stats['keyspace_misses']
hit_rate = hits / (hits + misses) * 100 if (hits + misses) > 0 else 0
print(f'Cache hit rate: {hit_rate:.1f}%')
```


## Resources

- [INFO Command](https://redis.io/docs/latest/commands/info/) — Redis INFO command reference

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*