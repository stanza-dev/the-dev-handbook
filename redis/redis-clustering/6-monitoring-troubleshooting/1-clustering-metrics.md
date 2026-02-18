---
source_course: "redis-clustering"
source_lesson: "redis-clustering-metrics"
---

# Key Metrics and Health Indicators

Monitor the right metrics to maintain a healthy Redis cluster.

## Essential Metrics

### Memory Metrics

```redis
# Get memory information
INFO memory

# Key metrics:
# used_memory: Total allocated memory
# used_memory_rss: Memory from OS perspective
# mem_fragmentation_ratio: RSS / used_memory
# maxmemory: Configured limit
```

```
┌─────────────────────────────────────────────────────────────┐
│  Metric                    Alert Threshold    Action        │
├─────────────────────────────────────────────────────────────┤
│  used_memory/maxmemory     > 80%             Warn           │
│  used_memory/maxmemory     > 95%             Critical       │
│  mem_fragmentation_ratio   > 1.5             Investigate    │
│  mem_fragmentation_ratio   < 1.0             Swapping!      │
└─────────────────────────────────────────────────────────────┘
```

### Performance Metrics

```redis
INFO stats

# Key metrics:
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

# Replica perspective:
# master_link_status: up/down
# master_last_io_seconds_ago: Time since last communication
# slave_repl_offset: Replication position
```

```python
def check_replication_health(r):
    """Monitor replication lag"""
    info = r.info('replication')
    
    if info['role'] == 'master':
        for i in range(info.get('connected_slaves', 0)):
            slave_info = info.get(f'slave{i}', '')
            # Parse lag from slave info
            lag = int(slave_info.split('lag=')[1].split(',')[0])
            if lag > 10:
                alert(f"Replica {i} lag: {lag} seconds")
    
    elif info['role'] == 'slave':
        if info.get('master_link_status') != 'up':
            alert("Replica disconnected from master!")
```

### Cluster-Specific Metrics

```redis
CLUSTER INFO

# Key metrics:
# cluster_state: ok/fail
# cluster_slots_assigned: Should be 16384
# cluster_slots_ok: Slots with active masters
# cluster_known_nodes: Total nodes in cluster
```

## Monitoring Tools

### Redis CLI Monitoring

```bash
# Real-time command monitoring
redis-cli MONITOR

# Caution: High overhead in production!

# Better: Sample slow queries
redis-cli SLOWLOG GET 10
```

### Prometheus Metrics

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

```python
# Key Prometheus queries
queries = {
    'hit_rate': 'rate(redis_keyspace_hits_total[5m]) / (rate(redis_keyspace_hits_total[5m]) + rate(redis_keyspace_misses_total[5m]))',
    'memory_usage': 'redis_memory_used_bytes / redis_memory_max_bytes',
    'ops_per_second': 'rate(redis_commands_processed_total[1m])',
    'replication_lag': 'redis_connected_slave_lag_seconds'
}
```

📖 [INFO Command](https://redis.io/docs/latest/commands/info/)

## Resources

- [INFO Command](https://redis.io/docs/latest/commands/info/) — Redis INFO command reference

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*