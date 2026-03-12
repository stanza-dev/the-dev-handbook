---
source_course: "redis-clustering"
source_lesson: "redis-clustering-sentinel-client-integration"
---

# Sentinel Client Integration and Best Practices

## Introduction

Sentinel only delivers high availability if your application clients are properly integrated. A correctly configured client automatically follows master changes during failover with minimal disruption.

## Key Concepts

- **Sentinel-aware Client**: A client library that queries Sentinel for the current master address instead of connecting to a hardcoded IP
- **+switch-master Event**: The Pub/Sub event Sentinel publishes when a new master is promoted
- **Connection Retry**: The client behavior of reconnecting to the new master after receiving a connection error
- **ReadOnlyError**: The error returned when a client sends a write to a demoted master that is now a replica

## Real World Context

Even with a perfectly configured Sentinel cluster, applications experience downtime if clients are not Sentinel-aware. Hardcoded master IPs mean manual restarts after every failover. Proper client integration is what turns Sentinel from a monitoring tool into a true HA solution.

## Deep Dive

### Client Library Support

Most popular Redis client libraries support Sentinel natively:

| Language | Library | Sentinel Support |
|----------|---------|------------------|
| Python | redis-py | `Sentinel` class |
| Node.js | ioredis | `sentinels` option |
| Java | Jedis | `JedisSentinelPool` |
| Go | go-redis | `NewFailoverClient` |

### Python Client with Full Error Handling

```python
import redis
from redis.sentinel import Sentinel
import time

def create_sentinel_connection():
    sentinel = Sentinel([
        ('sentinel-1.example.com', 26379),
        ('sentinel-2.example.com', 26379),
        ('sentinel-3.example.com', 26379),
    ], socket_timeout=0.5, password='sentinel_password')
    return sentinel

def get_master_with_retry(sentinel, service_name='mymaster', retries=3):
    for attempt in range(retries):
        try:
            master = sentinel.master_for(
                service_name,
                socket_timeout=0.5,
                password='redis_password',
                retry_on_timeout=True
            )
            master.ping()
            return master
        except redis.ConnectionError:
            if attempt < retries - 1:
                time.sleep(1 * (attempt + 1))
            else:
                raise
```

### Node.js with ioredis

```javascript
const Redis = require('ioredis');

const redis = new Redis({
  sentinels: [
    { host: 'sentinel-1.example.com', port: 26379 },
    { host: 'sentinel-2.example.com', port: 26379 },
    { host: 'sentinel-3.example.com', port: 26379 },
  ],
  name: 'mymaster',
  sentinelPassword: 'sentinel_password',
  password: 'redis_password',
  retryStrategy(times) {
    return Math.min(times * 100, 3000);
  },
});
```

### Pub/Sub Events for Monitoring

Sentinel publishes events your application can subscribe to:

```python
import redis

def monitor_sentinel_events():
    r = redis.Redis(host='sentinel-1.example.com', port=26379,
                    password='sentinel_password')
    pubsub = r.pubsub()
    pubsub.psubscribe('*')
    for message in pubsub.listen():
        if message['type'] == 'pmessage':
            channel = message['channel'].decode()
            data = message['data'].decode()
            if '+switch-master' in channel:
                print(f'Master changed: {data}')
```

### Connection Handling During Failover

During a failover (typically 5-15 seconds), clients experience:
1. Connection errors to the old master
2. Brief period where no master is available
3. READONLY errors if connected to a replica that has not yet been promoted

The sentinel client automatically retries on the new master when configured with `retry_on_timeout=True`.

## Common Pitfalls

1. **Using a single Sentinel address** -- If that Sentinel is down, your client cannot discover the master. Always provide all Sentinel addresses to the client.
2. **Not handling ReadOnlyError** -- During failover, your client may briefly connect to a server that has been demoted to replica. Catch `ReadOnlyError` and re-query Sentinel for the new master.

## Best Practices

1. **Implement exponential backoff for reconnection** -- During failover, hammering Sentinel with connection attempts causes unnecessary load. Use increasing delays between retries.
2. **Separate read and write connections** -- Use `master_for()` for writes and `slave_for()` for reads. This automatically routes traffic and provides read scaling.

## Summary

- Use Sentinel-aware client libraries instead of hardcoded master addresses
- Handle ReadOnlyError and ConnectionError for seamless failover
- Subscribe to Sentinel Pub/Sub events for real-time failover visibility
- Provide all Sentinel addresses to clients for redundancy
- Implement retry logic with exponential backoff during failover windows

## Code Examples

**Sentinel-aware client with separate read/write connections**

```python
from redis.sentinel import Sentinel

sentinel = Sentinel([
    ('sentinel-1.example.com', 26379),
    ('sentinel-2.example.com', 26379),
    ('sentinel-3.example.com', 26379),
], socket_timeout=0.5, password='sentinel_password')

# Write to master
master = sentinel.master_for('mymaster', password='redis_password')
master.set('key', 'value')

# Read from replica
replica = sentinel.slave_for('mymaster', password='redis_password')
value = replica.get('key')
```


## Resources

- [Redis Sentinel Client Guidelines](https://redis.io/docs/latest/operate/oss_and_stack/management/sentinel/) — Guidelines for connecting to Redis through Sentinel

---

> 📘 *This lesson is part of the [Clustering and High Availability Architecture](https://stanza.dev/courses/redis-clustering) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*