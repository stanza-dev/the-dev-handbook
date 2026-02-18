---
source_course: "redis-probabilistic"
source_lesson: "redis-probabilistic-tdigest"
---

# t-digest: Percentile Estimation

t-digest accurately estimates percentiles (median, 95th, 99th) of a data stream using fixed memory.

## The Problem

```
┌─────────────────────────────────────────────────────────────┐
│  Problem: Find the 99th percentile response time           │
│                                                             │
│  Exact: Store all values → Sort → Find position            │
│  Memory: O(n) - grows with data                            │
│                                                             │
│  t-digest: Streaming percentile estimation                 │
│  Memory: O(1) - fixed compression parameter                │
└─────────────────────────────────────────────────────────────┘
```

## Basic Commands

### TDIGEST.CREATE: Create Structure

```redis
# Create t-digest with default compression (100)
TDIGEST.CREATE latency

# With custom compression (higher = more accurate, more memory)
TDIGEST.CREATE latency COMPRESSION 500
```

### TDIGEST.ADD: Add Values

```redis
# Add single value
TDIGEST.ADD latency 45.2

# Add multiple values
TDIGEST.ADD latency 45.2 52.1 38.7 61.3 49.8

# Add with weights (for pre-aggregated data)
TDIGEST.ADD latency 45.2 10 52.1 5
# 45.2 appeared 10 times, 52.1 appeared 5 times
```

### TDIGEST.QUANTILE: Get Percentiles

```redis
# Get median (50th percentile)
TDIGEST.QUANTILE latency 0.5
# Returns: [48.5]

# Get multiple percentiles
TDIGEST.QUANTILE latency 0.5 0.95 0.99
# Returns: [48.5, 95.2, 128.7]

# Common SLA percentiles
TDIGEST.QUANTILE latency 0.5 0.9 0.95 0.99 0.999
```

### TDIGEST.CDF: Cumulative Distribution

```redis
# What percentile is value 50?
TDIGEST.CDF latency 50
# Returns: [0.52] (52% of values are ≤ 50)

# Multiple values
TDIGEST.CDF latency 50 100 150
# Returns: [0.52, 0.89, 0.97]
```

### Other Commands

```redis
# Get min and max values
TDIGEST.MIN latency
TDIGEST.MAX latency

# Get trimmed mean (ignoring outliers)
TDIGEST.TRIMMED_MEAN latency 0.1 0.9
# Mean excluding bottom 10% and top 10%

# Merge multiple digests
TDIGEST.MERGE combined COMPRESSION 200 2 latency1 latency2
```

## Practical Example: Response Time Monitoring

```python
import redis
import time

r = redis.Redis()

class LatencyMonitor:
    def __init__(self, name, compression=200):
        self.key = f'latency:{name}'
        try:
            r.execute_command('TDIGEST.CREATE', self.key, 'COMPRESSION', compression)
        except redis.ResponseError:
            pass  # Already exists
    
    def record(self, latency_ms):
        """Record a latency measurement"""
        r.execute_command('TDIGEST.ADD', self.key, latency_ms)
    
    def get_percentiles(self):
        """Get standard SLA percentiles"""
        result = r.execute_command(
            'TDIGEST.QUANTILE', self.key,
            0.5, 0.9, 0.95, 0.99
        )
        return {
            'p50': result[0],
            'p90': result[1],
            'p95': result[2],
            'p99': result[3]
        }
    
    def get_stats(self):
        """Get min, max, and percentiles"""
        min_val = r.execute_command('TDIGEST.MIN', self.key)
        max_val = r.execute_command('TDIGEST.MAX', self.key)
        percentiles = self.get_percentiles()
        return {
            'min': min_val,
            'max': max_val,
            **percentiles
        }

# Usage
monitor = LatencyMonitor('api:endpoint')

# Record latencies
for latency in [45, 52, 48, 120, 55, 42, 67, 51]:
    monitor.record(latency)

print(monitor.get_percentiles())
# {'p50': 51.5, 'p90': 100.0, 'p95': 120.0, 'p99': 120.0}
```

## Time-Windowed Percentiles

```python
from datetime import datetime

def record_latency_windowed(latency):
    """Record latency in current minute's t-digest"""
    minute = datetime.now().strftime('%Y-%m-%d:%H:%M')
    key = f'latency:{minute}'
    
    # Create if not exists (will error if exists, that's fine)
    try:
        r.execute_command('TDIGEST.CREATE', key)
        r.expire(key, 3600)  # Keep for 1 hour
    except:
        pass
    
    r.execute_command('TDIGEST.ADD', key, latency)

def get_hourly_percentiles():
    """Merge last 60 minutes and get percentiles"""
    keys = []
    now = datetime.now()
    for i in range(60):
        minute = (now - timedelta(minutes=i)).strftime('%Y-%m-%d:%H:%M')
        keys.append(f'latency:{minute}')
    
    # Filter to existing keys
    existing = [k for k in keys if r.exists(k)]
    
    if not existing:
        return None
    
    # Merge into temporary key
    r.execute_command('TDIGEST.MERGE', 'latency:temp', len(existing), *existing)
    result = r.execute_command('TDIGEST.QUANTILE', 'latency:temp', 0.5, 0.95, 0.99)
    r.delete('latency:temp')
    
    return result
```

📖 [t-digest Commands](https://redis.io/docs/latest/develop/data-types/probabilistic/t-digest/)

## Resources

- [t-digest](https://redis.io/docs/latest/develop/data-types/probabilistic/t-digest/) — Redis t-digest documentation

---

> 📘 *This lesson is part of the [Probabilistic Data Structures and Analytics](https://stanza.dev/courses/redis-probabilistic) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*