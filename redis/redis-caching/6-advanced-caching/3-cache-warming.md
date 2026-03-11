---
source_course: "redis-caching"
source_lesson: "redis-caching-cache-warming"
---

# Cache Warming and Preloading

## Introduction
A cold cache — one with no data — means every initial request is a cache miss that hits the database. Cache warming pre-populates Redis with data before traffic arrives, ensuring fast responses from the very first request after a deployment or restart.

## Key Concepts
- **Cold Cache**: A cache with no entries, causing 100% miss rate until populated.
- **Warm Cache**: A cache pre-loaded with expected data, providing hits from the start.
- **Lazy Warming**: Data is cached only on first access (default cache-aside behavior).
- **Eager Warming**: Data is proactively loaded into the cache before any request needs it.

## Real World Context
After deploying a new version of your application, all Redis data might be flushed or your instance restarted. If your site serves 10,000 requests per second, a cold cache means all 10,000 requests hit the database simultaneously — a recipe for an outage. Warming the cache during deployment prevents this.

## Deep Dive

### Strategy 1: Startup Warming

Load critical data when the application boots:

```python
import redis

def warm_cache_on_startup(r, db):
    """Load frequently accessed data at application start"""
    # Warm top products
    products = db.query("SELECT * FROM products WHERE featured = true")
    pipe = r.pipeline(transaction=False)
    for product in products:
        pipe.set(f"cache:product:{product.id}", serialize(product), ex=3600)
    pipe.execute()
    print(f"Warmed {len(products)} products")
    
    # Warm configuration
    config = db.query("SELECT * FROM app_config")
    for item in config:
        r.set(f"cache:config:{item.key}", item.value, ex=86400)
    print(f"Warmed {len(config)} config entries")
```

Using a pipeline for bulk warming is critical — loading 10,000 keys one by one takes seconds, while a pipeline completes in milliseconds.

### Strategy 2: Scheduled Warming

Refresh cache on a schedule before data goes stale:

```python
import schedule
import time

def refresh_popular_products(r, db):
    """Run every 10 minutes to keep popular products warm"""
    products = db.query(
        "SELECT * FROM products ORDER BY views DESC LIMIT 1000"
    )
    pipe = r.pipeline(transaction=False)
    for p in products:
        pipe.set(f"cache:product:{p.id}", serialize(p), ex=900)
    pipe.execute()

schedule.every(10).minutes.do(refresh_popular_products, r, db)
```

This ensures the top 1,000 products are always cached, even if no one has accessed them recently.

### Strategy 3: Deploy-Time Warming

Warm the cache as part of your deployment pipeline:

```bash
# In your CI/CD pipeline:
# 1. Deploy new version
# 2. Run cache warming script
# 3. Switch traffic to new version

python warm_cache.py --categories --featured-products --config
```

By warming before switching traffic, users never experience a cold cache.

### Selective Warming

Do not warm everything — focus on data with the highest impact:

```python
def selective_warm(r, db):
    """Only warm data that is expensive and frequently accessed"""
    # High impact: frequently accessed + expensive to compute
    warm_analytics_dashboards(r, db)     # 500ms DB query, 100 req/min
    warm_product_recommendations(r, db)  # 2s ML inference, 50 req/min
    
    # Skip: cheap queries or rarely accessed data
    # warm_user_settings(r, db)  # 5ms query, 1 req/day per user
```

## Common Pitfalls
1. **Warming too much data** — Loading your entire database into Redis wastes memory and time. Focus on the top 1-5% of data by access frequency.
2. **Not using pipelining for bulk loads** — Warming 10,000 keys without pipelining takes 100x longer due to network round-trips.

## Best Practices
1. **Warm before traffic switches** — In blue-green deployments, warm the new environment's cache before routing traffic to it.
2. **Log warming metrics** — Track how many keys were warmed and how long it took. Alert if warming fails or takes too long.

## Summary
- A cold cache causes a burst of database load after deploys or restarts
- Eager warming pre-loads critical data using pipelines for efficiency
- Scheduled warming keeps popular data fresh between access windows
- Focus warming on high-impact data: frequently accessed and expensive to generate

## Code Examples

**Cache warming with pipelining — bulk-loads top products into Redis at startup**

```python
import redis

def warm_cache(r, db):
    """Warm top 100 products using pipeline"""
    products = db.query(
        "SELECT * FROM products ORDER BY views DESC LIMIT 100"
    )
    pipe = r.pipeline(transaction=False)
    for p in products:
        pipe.set(f"cache:product:{p.id}", serialize(p), ex=3600)
    pipe.execute()
    print(f"Warmed {len(products)} products")
```


## Resources

- [Redis Pipelining](https://redis.io/docs/latest/develop/use/pipelining/) — Using pipelining for efficient bulk operations

---

> 📘 *This lesson is part of the [High-Performance Caching Strategies](https://stanza.dev/courses/redis-caching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*