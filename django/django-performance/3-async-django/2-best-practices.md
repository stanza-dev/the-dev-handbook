---
source_course: "django-performance"
source_lesson: "django-performance-async-best-practices"
---

# Async Django Best Practices

## Introduction

Async Django requires careful consideration of when async provides benefits and when it adds complexity without gains.

## Key Concepts

**I/O Bound**: Operations waiting for external resources (network, disk).

**CPU Bound**: Operations using processor time.

## Real World Context

This topic directly impacts production application performance. Teams that master these techniques reduce page load times, lower infrastructure costs, and deliver better user experiences.

## Deep Dive

> **Important**: While Django's ORM exposes async methods (prefixed with `a`, e.g., `aget()`, `afilter()`), the underlying database drivers (psycopg2, mysqlclient) may still execute synchronously under the hood. Django wraps synchronous database calls with `sync_to_async()` internally. True async database access requires an async-native driver like psycopg3. Always measure actual concurrency gains in your specific setup.

### When to Use Async

```python
# GOOD: Multiple I/O operations
async def dashboard(request):
    # Run concurrently - total time = max(time1, time2, time3)
    user_data, notifications, stats = await asyncio.gather(
        fetch_user_data(request.user),
        fetch_notifications(request.user),
        fetch_stats(request.user),
    )
    return render(request, 'dashboard.html', {...})

# BAD: Single database query (no benefit)
async def article_detail(request, pk):
    article = await Article.objects.aget(pk=pk)
    return render(request, 'detail.html', {'article': article})
```

### Avoiding Common Pitfalls

```python
# BAD: Blocking call in async view
async def bad_view(request):
    import requests
    data = requests.get('https://api.example.com')  # Blocks!
    return JsonResponse(data.json())

# GOOD: Use async HTTP client
async def good_view(request):
    async with httpx.AsyncClient() as client:
        response = await client.get('https://api.example.com')
    return JsonResponse(response.json())
```

### Testing Async Views

```python
from django.test import AsyncClient

class AsyncViewTests(TestCase):
    async def test_async_view(self):
        client = AsyncClient()
        response = await client.get('/async-endpoint/')
        self.assertEqual(response.status_code, 200)
```

## Common Pitfalls

1. **Premature optimization** — Always profile before optimizing. Fix the biggest bottleneck first rather than guessing.
2. **Ignoring trade-offs** — Every optimization has costs. Caching adds complexity, indexes slow writes, and async adds cognitive overhead.

## Best Practices

1. **Use async for multiple I/O operations**: Concurrent external calls.
2. **Don't mix sync/async carelessly**: Understand the boundaries.
3. **Use httpx for HTTP calls**: async-native HTTP client.

## Summary

Async shines with concurrent I/O operations. Don't use async for single database queries. Use httpx instead of requests in async code. Test with AsyncClient.

## Code Examples

**Using asyncio.gather to run multiple I/O operations concurrently**

```python
# GOOD: Multiple I/O operations
async def dashboard(request):
    # Run concurrently - total time = max(time1, time2, time3)
    user_data, notifications, stats = await asyncio.gather(
        fetch_user_data(request.user),
        fetch_notifications(request.user),
        fetch_stats(request.user),
    )
    return render(request, 'dashboard.html', {...})

# BAD: Single database query (no benefit)
async def article_detail(request, pk):
    article = await Article.objects.aget(pk=pk)
    return render(request, 'detail.html', {'article': article})
```


## Resources

- [Async Support](https://docs.djangoproject.com/en/6.0/topics/async/) — Django async documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*