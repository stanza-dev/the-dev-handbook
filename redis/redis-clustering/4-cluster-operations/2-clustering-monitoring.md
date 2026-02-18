---
source_course: "redis-clustering"
source_lesson: "redis-clustering-monitoring"
---

# Monitoring Redis Deployments

Effective monitoring is essential for production Redis.

## Key Metrics to Monitor

### Memory

```redis
INFO memory

# Critical metrics:
# used_memory: Current memory usage
# used_memory_rss: OS-reported memory
# maxmemory: Configured limit
# mem_fragmentation_ratio: RSS/used (should be ~1.0-1.5)
```

**Alerts**:
- used_memory > 80% of maxmemory (warning)
- used_memory > 95% of maxmemory (critical)
- mem_fragmentation_ratio > 2.0 (high fragmentation)

### Performance

```redis
INFO stats

# Key metrics:
# instantaneous_ops_per_sec: Current throughput
# total_commands_processed: Cumulative commands
# keyspace_hits/misses: Cache efficiency
```

**Calculate hit rate**:
```
Hit Rate = keyspace_hits / (keyspace_hits + keyspace_misses)
```

### Replication

```redis
INFO replication

# Check:
# master_link_status: Should be 'up'
# master_last_io_seconds_ago: Replication lag
# connected_slaves: Number of replicas
```

**Alerts**:
- master_link_status != up
- master_last_io_seconds_ago > 30 (high lag)
- connected_slaves < expected

### Clients

```redis
INFO clients

# Metrics:
# connected_clients: Current connections
# blocked_clients: Clients waiting (BLPOP, etc.)
# client_recent_max_input_buffer: Connection buffer usage
```

### Slow Queries

```redis
# Enable slow log
CONFIG SET slowlog-log-slower-than 10000  # 10ms
CONFIG SET slowlog-max-len 128

# View slow queries
SLOWLOG GET 10

# Returns:
# 1) (integer) 14        # ID
# 2) (integer) 1704067200 # Timestamp
# 3) (integer) 15234     # Duration (microseconds)
# 4) 1) "KEYS"           # Command
#    2) "*"
```

## Monitoring Commands

### MONITOR (Debugging)

```redis
# Real-time command stream (USE SPARINGLY!)
MONITOR

# Shows all commands:
# 1704067200.123456 [0 127.0.0.1:54321] "SET" "key" "value"
```

**Warning**: MONITOR has significant performance impact.

### CLIENT LIST

```redis
# View all connected clients
CLIENT LIST

# Kill problematic client
CLIENT KILL ID 5
```

## Alerting Thresholds

| Metric | Warning | Critical |
|--------|---------|----------|
| Memory usage | 80% | 95% |
| CPU usage | 70% | 90% |
| Replication lag | 10s | 60s |
| Connected clients | 80% of max | 95% of max |
| Cache hit rate | <90% | <70% |
| Slow queries/sec | >10 | >50 |

## Health Check Endpoint

```python
from flask import Flask, jsonify
import redis

app = Flask(__name__)
r = redis.Redis()

@app.route('/health')
def health_check():
    try:
        # Basic connectivity
        r.ping()
        
        # Memory check
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

📖 [Redis Monitoring](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/latency-monitor/)

## Resources

- [Latency Monitoring](https://redis.io/docs/latest/operate/oss_and_stack/management/optimization/latency-monitor/) — Redis latency monitoring and debugging

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*