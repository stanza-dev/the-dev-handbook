---
source_course: "django-performance"
source_lesson: "django-performance-async-views"
---

# Async Views and Middleware

## Introduction

Django 6.0 provides an async-compatible interface for views, middleware, and ORM operations. Async views excel when your application needs to make multiple concurrent I/O operations — such as calling external APIs, querying multiple databases, or handling WebSocket connections. Note that while the ORM exposes async methods (prefixed with 'a'), the underlying database drivers may still execute synchronously.

## Key Concepts

- **Async View**: A view function defined with `async def` that Django runs in an async context.
- **ASGI**: Asynchronous Server Gateway Interface — the async equivalent of WSGI, required for async Django.
- **sync_to_async / async_to_sync**: Bridge functions from `asgiref` that let you call sync code from async contexts and vice versa.
- **Async ORM**: Django ORM methods prefixed with 'a' (e.g., `aget()`, `acreate()`) that can be awaited in async views.

## Real World Context

Async views shine in dashboard applications that aggregate data from multiple sources. Instead of sequentially calling a user API, a notifications API, and a stats API (taking the sum of all response times), you can use `asyncio.gather()` to call all three concurrently (taking only the time of the slowest call). A dashboard that took 3 seconds with sync views can respond in 1 second with async.

## Deep Dive

Django 6 uses `asgiref.sync.iscoroutinefunction` (not `asyncio.iscoroutinefunction`) to detect async views and middleware. You must also call `markcoroutinefunction(self)` in `__init__` so Django recognizes your middleware as async-capable.


## Introduction

Django 6.0 provides an async-compatible interface for views, middleware, and ORM operations. Note that while the ORM exposes async methods (prefixed with 'a'), the underlying database drivers may still execute synchronously under the hood, allowing better handling of concurrent requests.

## Async Views

```python
from django.http import JsonResponse
import asyncio
import httpx


async def async_article_list(request):
    """Async view function."""
    articles = []
    async for article in Article.objects.filter(status='published'):
        articles.append({
            'title': article.title,
            'slug': article.slug,
        })
    return JsonResponse({'articles': articles})


async def fetch_external_data(request):
    """Fetch data from external APIs concurrently."""
    async with httpx.AsyncClient() as client:
        # Fetch multiple APIs concurrently
        responses = await asyncio.gather(
            client.get('https://api.example.com/news'),
            client.get('https://api.example.com/weather'),
            client.get('https://api.example.com/stocks'),
        )
    
    return JsonResponse({
        'news': responses[0].json(),
        'weather': responses[1].json(),
        'stocks': responses[2].json(),
    })
```

## Async Class-Based Views

```python
from django.views import View
from django.http import JsonResponse


class AsyncArticleView(View):
    async def get(self, request, slug):
        article = await Article.objects.aget(slug=slug)
        return JsonResponse({
            'title': article.title,
            'body': article.body,
        })
    
    async def post(self, request, slug):
        article = await Article.objects.aget(slug=slug)
        article.views += 1
        await article.asave()
        return JsonResponse({'views': article.views})
```

## Async ORM Operations

```python
# Django 6.0 async ORM methods

# Get single object
article = await Article.objects.aget(pk=1)
article = await Article.objects.afirst()
article = await Article.objects.aget_or_create(title='New')

# Check existence
exists = await Article.objects.filter(status='published').aexists()
count = await Article.objects.filter(status='published').acount()

# Iterate
async for article in Article.objects.filter(status='published'):
    print(article.title)

# Collect to list
articles = [a async for a in Article.objects.all()]

# Save and delete
await article.asave()
await article.adelete()

# Bulk operations
await Article.objects.filter(status='draft').aupdate(status='archived')
await Article.objects.filter(status='archived').adelete()

# Aggregates
from django.db.models import Count, Avg
stats = await Article.objects.aaggregate(total=Count('id'), avg_views=Avg('views'))
```

## Mixing Sync and Async

```python
from asgiref.sync import sync_to_async, async_to_sync


# Call sync code from async context
@sync_to_async
def sync_external_api_call():
    """Sync function that can't be async."""
    import requests
    return requests.get('https://api.example.com/data').json()


async def async_view(request):
    # Use sync function in async context
    data = await sync_to_async_external_api_call()
    return JsonResponse(data)


# Call async code from sync context
def sync_view(request):
    data = async_to_sync(async_external_call)()
    return JsonResponse(data)
```

## Async Middleware

```python
class AsyncTimingMiddleware:
    async_capable = True
    sync_capable = True
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    async def __call__(self, request):
        import time
        start = time.time()
        
        # Handle async or sync response
        if iscoroutinefunction  # from asgiref.sync, NOT asyncio(self.get_response):
            response = await self.get_response(request)
        else:
            response = self.get_response(request)
        
        duration = time.time() - start
        response['X-Request-Duration'] = f'{duration:.3f}s'
        return response
```

## Running Async Django

```python
# Use ASGI server (uvicorn, daphne, hypercorn)
# uvicorn myproject.asgi:application

# asgi.py
import os
from django.core.asgi import get_asgi_application

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

application = get_asgi_application()
```

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Using `requests` library in async views** — The `requests` library is synchronous and will block the entire event loop. Use `httpx` with `AsyncClient` instead for HTTP calls in async views.
2. **Expecting async to speed up single queries** — A single `await Article.objects.aget(pk=1)` is not faster than its sync equivalent. Async benefits come from concurrent operations, not individual ones.
3. **Forgetting to use the ASGI server** — Async views require an ASGI server (Uvicorn, Daphne). Running them under WSGI (Gunicorn with sync workers) silently falls back to synchronous execution.

## Best Practices

1. **Use async only when you have concurrent I/O** — If a view makes a single database query, keep it synchronous. Async adds complexity without benefit for single operations.
2. **Use `httpx.AsyncClient` for external API calls** — It provides the same ergonomics as `requests` but works natively in async contexts.
3. **Test with `AsyncClient`** — Django's `AsyncClient` in tests properly handles async views and middleware.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Async view fetching data from multiple external APIs concurrently**

```python
from django.http import JsonResponse
import asyncio
import httpx


async def async_article_list(request):
    """Async view function."""
    articles = []
    async for article in Article.objects.filter(status='published'):
        articles.append({
            'title': article.title,
            'slug': article.slug,
        })
    return JsonResponse({'articles': articles})


async def fetch_external_data(request):
    """Fetch data from external APIs concurrently."""
    async with httpx.AsyncClient() as client:
        # Fetch multiple APIs concurrently
# ...
```


## Resources

- [Async Support](https://docs.djangoproject.com/en/6.0/topics/async/) — Django async documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*