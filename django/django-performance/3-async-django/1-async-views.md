---
source_course: "django-performance"
source_lesson: "django-performance-async-views"
---

# Async Views and Middleware

Django 6.0 has full async support, allowing better handling of concurrent requests.

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
        if asyncio.iscoroutinefunction(self.get_response):
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

## Resources

- [Async Support](https://docs.djangoproject.com/en/6.0/topics/async/) — Django async documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*