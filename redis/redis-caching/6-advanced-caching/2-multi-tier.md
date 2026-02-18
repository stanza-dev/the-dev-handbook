---
source_course: "redis-caching"
source_lesson: "redis-caching-multi-tier"
---

# Multi-Tier Caching

Combine multiple cache layers for optimal performance and cost efficiency.

## Cache Hierarchy

```
┌─────────────────────────────────────────────────────────────┐
│  Tier 1: Application Memory (fastest, smallest)             │
│          Response time: <1ms, Size: MBs                     │
├─────────────────────────────────────────────────────────────┤
│  Tier 2: Redis (fast, medium)                               │
│          Response time: 1-5ms, Size: GBs                    │
├─────────────────────────────────────────────────────────────┤
│  Tier 3: Database (slow, largest)                           │
│          Response time: 10-100ms, Size: TBs                 │
└─────────────────────────────────────────────────────────────┘
```

## Implementation Pattern

```python
class MultiTierCache:
    def __init__(self, redis_client, local_cache_size=1000):
        self.redis = redis_client
        self.local = LRUCache(local_cache_size)
    
    def get(self, key, fetch_func, local_ttl=60, redis_ttl=300):
        # Tier 1: Check local cache
        data = self.local.get(key)
        if data is not None:
            return data
        
        # Tier 2: Check Redis
        data = self.redis.get(key)
        if data is not None:
            # Populate local cache
            self.local.set(key, data, ttl=local_ttl)
            return data
        
        # Tier 3: Fetch from database
        data = fetch_func()
        
        # Populate both caches
        self.redis.set(key, data, ex=redis_ttl)
        self.local.set(key, data, ttl=local_ttl)
        
        return data
    
    def invalidate(self, key):
        # Invalidate both tiers
        self.local.delete(key)
        self.redis.delete(key)
```

## Hot vs Cold Data Separation

Keep hot data in memory, cold data in Redis:

```python
class HotColdCache:
    def __init__(self, redis_client, hot_threshold=100):
        self.redis = redis_client
        self.hot_cache = {}  # In-memory for hot keys
        self.access_count = {}  # Track access frequency
        self.hot_threshold = hot_threshold
    
    def get(self, key, fetch_func):
        # Track access
        self.access_count[key] = self.access_count.get(key, 0) + 1
        
        # Check if hot (in-memory)
        if key in self.hot_cache:
            return self.hot_cache[key]
        
        # Check Redis
        data = self.redis.get(key)
        if data is None:
            data = fetch_func()
            self.redis.set(key, data, ex=300)
        
        # Promote to hot cache if accessed frequently
        if self.access_count[key] >= self.hot_threshold:
            self.hot_cache[key] = data
        
        return data
```

## CDN + Redis + Database

For web applications:

```
┌───────────────────────────────────────────────────────────┐
│  User Request                                              │
│       ↓                                                    │
│  CDN (static assets, edge-cached pages)                   │
│       ↓ (cache miss)                                      │
│  Application Server                                        │
│       ↓                                                    │
│  Redis (dynamic data cache)                               │
│       ↓ (cache miss)                                      │
│  Database                                                  │
└───────────────────────────────────────────────────────────┘
```

```python
def get_product_page(product_id):
    cache_key = f"page:product:{product_id}"
    
    # Check if CDN should cache (via headers)
    # Static products: Cache-Control: max-age=3600
    # Dynamic products: Cache-Control: private
    
    # Redis for dynamic data
    product = cache.get(
        f"product:{product_id}",
        lambda: db.get_product(product_id),
        redis_ttl=300
    )
    
    return render_product_page(product)
```

## Regional Cache Layers

For global applications:

```
┌───────────────────────────────────────────────────────────┐
│  US User → US Redis (local) → Global Redis → Database     │
│  EU User → EU Redis (local) → Global Redis → Database     │
│  Asia User → Asia Redis (local) → Global Redis → Database │
└───────────────────────────────────────────────────────────┘
```

**Benefits:**
- Lower latency for users in each region
- Reduced cross-region traffic
- Better fault tolerance

📖 [Caching Architectures](https://redis.io/docs/latest/develop/use/patterns/)

## Resources

- [Redis Patterns](https://redis.io/docs/latest/develop/use/patterns/) — Advanced caching patterns and architectures

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*