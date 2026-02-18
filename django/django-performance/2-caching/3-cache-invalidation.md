---
source_course: "django-performance"
source_lesson: "django-performance-cache-invalidation"
---

# Cache Invalidation Strategies

## Introduction

Cache invalidation is notoriously difficult. Poor invalidation leads to stale data or cache misses.

## Key Concepts

**TTL-Based**: Cache expires after time period.

**Event-Based**: Invalidate on data changes.

**Version-Based**: Change cache key on updates.

## Deep Dive

### Signal-Based Invalidation

```python
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver

@receiver([post_save, post_delete], sender=Article)
def invalidate_article_cache(sender, instance, **kwargs):
    cache.delete(f'article:{instance.slug}')
    cache.delete('article_list')
    cache.delete(f'author:{instance.author_id}:articles')
```

### Version-Based Keys

```python
class Article(models.Model):
    version = models.IntegerField(default=0)
    
    def save(self, *args, **kwargs):
        self.version += 1
        super().save(*args, **kwargs)
    
    @property
    def cache_key(self):
        return f'article:{self.pk}:v{self.version}'
```

### Cache Tags (with django-redis)

```python
from django_redis import get_redis_connection

def cache_with_tags(key, value, tags, timeout=3600):
    cache.set(key, value, timeout)
    r = get_redis_connection('default')
    for tag in tags:
        r.sadd(f'tag:{tag}', key)

def invalidate_tag(tag):
    r = get_redis_connection('default')
    keys = r.smembers(f'tag:{tag}')
    if keys:
        cache.delete_many([k.decode() for k in keys])
    r.delete(f'tag:{tag}')

# Usage
cache_with_tags('article:123', article, ['articles', f'author:{author_id}'])
invalidate_tag('articles')  # Clears all article caches
```

## Best Practices

1. **Prefer TTL + event invalidation**: Belt and suspenders.
2. **Use cache prefixes**: Easier to invalidate by category.
3. **Log cache invalidations**: Helps debug stale data issues.

## Summary

Combine TTL with signal-based invalidation for reliability. Version-based keys avoid race conditions. Cache tags simplify bulk invalidation.

## Resources

- [Cache Invalidation](https://docs.djangoproject.com/en/6.0/topics/cache/#cache-versioning) — Cache versioning documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*