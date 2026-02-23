---
source_course: "django-performance"
source_lesson: "django-performance-async-orm-operations"
---

# Async ORM Operations

## Introduction

Django 6 provides async versions of all ORM operations, prefixed with `a` (e.g., `aget()`, `acreate()`, `acount()`). These allow you to write async views that interact with the database without blocking the event loop. However, the underlying database drivers may still execute synchronously under the hood.

## Key Concepts

**Async ORM Methods**: Prefixed with 'a' (aget, afilter, etc.).

**Async Iteration**: Use 'async for' with querysets.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Deep Dive

> **Important**: While Django's ORM exposes async methods (prefixed with `a`, e.g., `aget()`, `afilter()`), the underlying database drivers (psycopg2, mysqlclient) may still execute synchronously under the hood. Django wraps synchronous database calls with `sync_to_async()` internally. True async database access requires an async-native driver like psycopg3. Always measure actual concurrency gains in your specific setup.

### Async Query Methods

```python
# Single object
article = await Article.objects.aget(pk=1)
article = await Article.objects.afirst()

# Existence and count
exists = await Article.objects.filter(status='published').aexists()
count = await Article.objects.filter(status='published').acount()

# Create, update, delete
article = await Article.objects.acreate(title='New')
await Article.objects.filter(status='draft').aupdate(status='archived')
await article.adelete()

# Get or create
article, created = await Article.objects.aget_or_create(
    slug='my-article',
    defaults={'title': 'My Article'}
)
```

### Async Iteration

```python
async def get_articles():
    articles = []
    async for article in Article.objects.filter(status='published'):
        articles.append({
            'title': article.title,
            'author': await article.author.aload(),
        })
    return articles

# Or use list comprehension
articles = [a async for a in Article.objects.all()[:10]]
```

### Async Aggregation

```python
from django.db.models import Count, Avg

stats = await Article.objects.aaggregate(
    total=Count('id'),
    avg_views=Avg('views')
)
```

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Use async for concurrent queries**: Multiple independent queries.
2. **Don't await in loops unnecessarily**: Use async for instead.
3. **Combine with asyncio.gather**: For parallel operations.

## Summary

Async ORM methods are prefixed with 'a'. Use async for for iteration. Combine with asyncio.gather for concurrent database operations.

## Code Examples

**Async ORM methods — aget, afirst, acount, acreate prefixed with a**

```python
# Single object
article = await Article.objects.aget(pk=1)
article = await Article.objects.afirst()

# Existence and count
exists = await Article.objects.filter(status='published').aexists()
count = await Article.objects.filter(status='published').acount()

# Create, update, delete
article = await Article.objects.acreate(title='New')
await Article.objects.filter(status='draft').aupdate(status='archived')
await article.adelete()

# Get or create
article, created = await Article.objects.aget_or_create(
    slug='my-article',
    defaults={'title': 'My Article'}
)
```


## Resources

- [Async ORM](https://docs.djangoproject.com/en/6.0/topics/db/queries/#async-queries) — Django async ORM documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*