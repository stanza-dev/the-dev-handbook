---
source_course: "django-performance"
source_lesson: "django-performance-cache-patterns"
---

# Caching Patterns and Strategies

## Introduction

Different caching strategies suit different use cases. Understanding patterns helps you choose the right approach.

## Key Concepts

**Cache-Aside**: Application manages cache reads/writes.

**Write-Through**: Updates cache on every write.

**TTL**: Time-to-live for cache entries.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Deep Dive

### Cache-Aside Pattern

```python
def get_article(slug):
    cache_key = f'article:{slug}'
    article = cache.get(cache_key)
    
    if article is None:
        article = Article.objects.get(slug=slug)
        cache.set(cache_key, article, timeout=3600)
    
    return article
```

### Write-Through Pattern

```python
class Article(models.Model):
    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        cache.set(f'article:{self.slug}', self, timeout=3600)
    
    def delete(self, *args, **kwargs):
        cache.delete(f'article:{self.slug}')
        super().delete(*args, **kwargs)
```

### Stale-While-Revalidate

```python
import threading

def get_with_stale(key, compute_fn, timeout=3600):
    value = cache.get(key)
    stale_key = f'{key}:stale'
    
    if value is None:
        # Check for stale value
        value = cache.get(stale_key)
        
        # Recompute in background
        thread = threading.Thread(
            target=lambda: cache.set(key, compute_fn(), timeout)
        )
        thread.start()
        
        if value is None:
            value = compute_fn()
            cache.set(key, value, timeout)
    
    # Always update stale cache
    cache.set(stale_key, value, timeout * 2)
    return value
```

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Use meaningful cache keys**: Include version for invalidation.
2. **Set appropriate TTLs**: Balance freshness vs performance.
3. **Handle cache failures gracefully**: Fall back to database.

## Summary

Cache-aside is simplest for read-heavy workloads. Write-through ensures consistency. Stale-while-revalidate provides best user experience for expensive computations.

## Code Examples

**Cache-aside pattern — check cache first, fall back to database**

```python
import threading

def get_with_stale(key, compute_fn, timeout=3600):
    value = cache.get(key)
    stale_key = f'{key}:stale'
    
    if value is None:
        # Check for stale value
        value = cache.get(stale_key)
        
        # Recompute in background
        thread = threading.Thread(
            target=lambda: cache.set(key, compute_fn(), timeout)
        )
        thread.start()
        
        if value is None:
            value = compute_fn()
            cache.set(key, value, timeout)
    
# ...
```


## Resources

- [Caching Strategies](https://docs.djangoproject.com/en/6.0/topics/cache/) — Django caching documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*