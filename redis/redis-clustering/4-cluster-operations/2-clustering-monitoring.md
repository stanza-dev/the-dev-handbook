---
source_course: "redis-clustering"
source_lesson: "redis-clustering-monitoring"
---

# Monitoring Redis Deployments

## Introduction

Effective monitoring is essential for production Redis. Knowing which metrics to watch, what thresholds to set, and how to build health checks prevents outages before they impact users.

## Key Concepts

- **INFO command**: Returns comprehensive server statistics organized by section (memory, stats, replication, clients)
- **SLOWLOG**: Records commands that exceed a configured execution time threshold
- **mem_fragmentation_ratio**: Ratio of OS-reported memory to Redis-reported memory; indicates memory health
- **keyspace_hits/misses**: Metrics for calculating cache hit rate

## Real World Context

Most Redis outages are preventable with proper monitoring. Memory exhaustion, connection limits, replication lag, and slow queries all produce warning signals well before they cause downtime.

## Deep Dive

### Memory Metrics

```redis
INFO memory
# used_memory: Current memory usage
# used_memory_rss: OS-reported memory
# maxmemory: Configured limit
# mem_fragmentation_ratio: RSS/used (should be ~1.0-1.5)
```

**Alerts:**
- used_memory > 80% of maxmemory (warning)
- used_memory > 95% of maxmemory (critical)
- mem_fragmentation_ratio > 2.0 (high fragmentation)

### Performance Metrics

```redis
INFO stats
# instantaneous_ops_per_sec: Current throughput
# total_commands_processed: Cumulative commands
# keyspace_hits/misses: Cache efficiency
```

**Calculate hit rate:**
```
Hit Rate = keyspace_hits / (keyspace_hits + keyspace_misses)
```

### Replication Metrics

```redis
INFO replication
# master_link_status: Should be 'up'
# master_last_io_seconds_ago: Replication lag
# connected_slaves: Number of replicas
```

**Alerts:**
- master_link_status != up
- master_last_io_seconds_ago > 30 (high lag)
- connected_slaves < expected

### Client Metrics

```redis
INFO clients
# connected_clients: Current connections
# blocked_clients: Clients waiting (BLPOP, etc.)
# client_recent_max_input_buffer: Connection buffer usage
```

### Slow Queries

```redis
CONFIG SET slowlog-log-slower-than 10000  # 10ms
CONFIG SET slowlog-max-len 128
SLOWLOG GET 10
```

### Monitoring Commands

```redis
# Real-time command stream (USE SPARINGLY - performance impact)
MONITOR

# View all connected clients
CLIENT LIST
```

### Alerting Thresholds

| Metric | Warning | Critical |
|--------|---------|----------|
| Memory usage | 80% | 95% |
| CPU usage | 70% | 90% |
| Replication lag | 10s | 60s |
| Connected clients | 80% of max | 95% of max |
| Cache hit rate | <90% | <70% |
| Slow queries/sec | >10 | >50 |

### Health Check Endpoint

```python
from flask import Flask, jsonify
import redis

app = Flask(__name__)
r = redis.Redis()

@app.route('/health')
def health_check():
    try:
        r.ping()
        info = r.info('memory')
        memory_usage = info['used_memory'] / info.get('maxmemory', float('inf'))
        if memory_usage > 0.95:
            return jsonify({'status': 'critical', 'memory': memory_usage}), 500
        elif memory_usage > 0.8:
            return jsonify({'status': 'warning', 'memory': memory_usage}), 200
        return jsonify({'status': 'healthy', 'memory': memory_usage}), 200
    except redis.ConnectionError:
        return jsonify({'status': 'down'}), 500
```

## Common Pitfalls

1. **Using MONITOR in production** -- MONITOR streams every command and has significant performance overhead (up to 50% throughput reduction). Use SLOWLOG instead for production debugging.
2. **Ignoring fragmentation ratio** -- A ratio below 1.0 means Redis is swapping to disk, causing severe latency. A ratio above 2.0 means wasted memory. Both require immediate attention.

## Best Practices

1. **Set up Prometheus + Grafana dashboards** -- Use redis-exporter to collect metrics and create dashboards for memory, throughput, replication lag, and slow queries.
2. **Implement health check endpoints** -- Every application that depends on Redis should have a health check that verifies connectivity and basic metrics.

## Summary

- Monitor memory, performance, replication, and client metrics via INFO command
- Set warning alerts at 80% memory and critical at 95%
- Use SLOWLOG to identify problematic queries without MONITOR overhead
- Watch mem_fragmentation_ratio: below 1.0 means swapping, above 2.0 means waste
- Build health check endpoints for automated monitoring

## Code Examples

**Key Redis monitoring commands for production health checks**

```bash
# Essential monitoring commands
redis-cli INFO memory
redis-cli INFO stats
redis-cli INFO replication
redis-cli SLOWLOG GET 10
```


## Resources

- [Latency Monitoring](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/latency-monitor/) — Redis latency monitoring and debugging

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*