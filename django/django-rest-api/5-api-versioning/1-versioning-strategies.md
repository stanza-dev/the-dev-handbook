---
source_course: "django-rest-api"
source_lesson: "django-rest-api-versioning-strategies"
---

# API Versioning Strategies

## Introduction

APIs are contracts. Once clients depend on your API's behavior, changing it can break their applications. Versioning gives you a way to evolve your API while keeping existing clients working.

Versioning prevents breaking changes from affecting existing clients. Choose a strategy that fits your needs.


## Key Concepts

**API Versioning**: The practice of maintaining multiple versions of your API simultaneously to prevent breaking changes.

**URL Path Versioning**: Including the version in the URL path (e.g., `/api/v1/articles/`). The most visible and common approach.

**Header Versioning**: Using the Accept header to specify the version (e.g., `Accept: application/vnd.myapi.v2+json`).

**Breaking Change**: Any change that requires existing clients to update their code, such as removing fields, changing response structure, or altering behavior.


## Real World Context

API versioning is critical for:
- **Public APIs**: Third-party developers build applications against your API
- **Mobile apps**: Users may not update immediately, so old versions must keep working
- **Enterprise integrations**: Partners have long development cycles and cannot adopt changes quickly
- **Microservices**: Internal services need stable contracts between teams

## URL Path Versioning

The most common and visible approach:

```python
# urls.py
from django.urls import path, include

urlpatterns = [
    path('api/v1/', include('myapp.api.v1.urls')),
    path('api/v2/', include('myapp.api.v2.urls')),
]
```

Organize your code:

```
myapp/
├── api/
│   ├── v1/
│   │   ├── __init__.py
│   │   ├── urls.py
│   │   ├── views.py
│   │   └── serializers.py
│   ├── v2/
│   │   ├── __init__.py
│   │   ├── urls.py
│   │   ├── views.py
│   │   └── serializers.py
│   └── common/
│       ├── __init__.py
│       └── utils.py
```

```python
# api/v1/views.py
def api_articles(request):
    articles = Article.objects.all()
    return JsonResponse({
        'articles': [{'id': a.id, 'title': a.title} for a in articles]
    })

# api/v2/views.py
def api_articles(request):
    articles = Article.objects.all()
    return JsonResponse({
        'data': [{
            'id': a.id,
            'type': 'article',
            'attributes': {
                'title': a.title,
                'body': a.body,
                'created_at': a.created_at.isoformat()
            }
        } for a in articles],
        'meta': {'version': 'v2'}
    })
```

V1 returns a simple flat list, while V2 adopts a more structured format with `type` and `attributes`. Both versions share the same model layer -- only the serialization differs.

## Header Versioning

Version in Accept header:

```python
# middleware.py
class APIVersionMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Parse Accept header: application/vnd.myapi.v2+json
        accept = request.headers.get('Accept', '')
        
        if 'vnd.myapi' in accept:
            import re
            match = re.search(r'vnd\.myapi\.(v\d+)', accept)
            if match:
                request.api_version = match.group(1)
            else:
                request.api_version = 'v1'
        else:
            request.api_version = 'v1'
        
        return self.get_response(request)
```

```python
# views.py
def api_articles(request):
    version = getattr(request, 'api_version', 'v1')
    
    articles = Article.objects.all()
    
    if version == 'v1':
        return serialize_v1(articles)
    elif version == 'v2':
        return serialize_v2(articles)
```

The middleware parses the `Accept` header to find version identifiers like `vnd.myapi.v2`, then attaches the version to the request object for views to inspect.

## Query Parameter Versioning

The simplest approach passes the version as a query parameter, though it is considered less RESTful since the version is not part of the resource representation:

```python
# Simple but less RESTful
def api_articles(request):
    version = request.GET.get('version', 'v1')
    # ...
```

Query parameter versioning is the simplest to implement but mixes API concerns into the URL query string.

## Version-Aware Views

This base class automatically routes requests to version-specific handler methods like `get_v1()` or `get_v2()`, falling back to the unversioned method:

```python
class VersionedAPIView:
    """Base class for versioned API views."""
    
    versions = ['v1', 'v2']
    default_version = 'v1'
    
    def get_version(self, request):
        # Check URL first
        if hasattr(request, 'resolver_match'):
            if 'version' in request.resolver_match.kwargs:
                return request.resolver_match.kwargs['version']
        
        # Then header
        return getattr(request, 'api_version', self.default_version)
    
    def dispatch(self, request, *args, **kwargs):
        version = self.get_version(request)
        
        if version not in self.versions:
            return JsonResponse(
                {'error': f'Unsupported API version: {version}'},
                status=400
            )
        
        # Call version-specific method if exists
        method = request.method.lower()
        handler_name = f'{method}_{version}'
        
        if hasattr(self, handler_name):
            handler = getattr(self, handler_name)
        else:
            handler = getattr(self, method, None)
        
        if handler:
            return handler(request, *args, **kwargs)
        
        return JsonResponse({'error': 'Method not allowed'}, status=405)


class ArticleAPIView(VersionedAPIView, View):
    def get_v1(self, request):
        """V1 response format."""
        articles = Article.objects.all()[:20]
        return JsonResponse({
            'articles': list(articles.values('id', 'title'))
        })
    
    def get_v2(self, request):
        """V2 response format with more details."""
        articles = Article.objects.select_related('author').all()[:20]
        return JsonResponse({
            'data': [{
                'id': a.id,
                'title': a.title,
                'author': a.author.username,
                'published': a.is_published
            } for a in articles]
        })
```

The `dispatch()` method checks for `get_v1` or `get_v2` method names first, so adding a new version is as simple as adding a new method to the subclass.

## Deprecation Strategy

Mark old API versions as deprecated by adding standard HTTP headers that inform clients about the upcoming removal:

```python
import warnings
from django.http import JsonResponse

def deprecated_api(view_func):
    """Mark an API version as deprecated."""
    def wrapper(request, *args, **kwargs):
        response = view_func(request, *args, **kwargs)
        
        # Add deprecation header
        if isinstance(response, JsonResponse):
            response['Deprecation'] = 'true'
            response['Sunset'] = 'Sat, 01 Jan 2025 00:00:00 GMT'
            response['Link'] = '</api/v2/>; rel="successor-version"'
        
        return response
    return wrapper


@deprecated_api
def api_v1_articles(request):
    # Old v1 endpoint
    pass
```

The `Sunset` header contains the removal date, and the `Link` header with `rel="successor-version"` points clients to the replacement endpoint.

## Common Pitfalls

1. **Too many active versions**: Maintaining more than 2-3 versions becomes a maintenance burden. Deprecate old versions aggressively.

2. **Versioning too often**: Create new versions only for breaking changes, not bug fixes or additive features.

3. **Duplicating code**: Share common logic between versions. Only the serialization and URL layers should differ.

## Best Practices

1. **Version from day one**: Even if you don't plan v2, starting with `/api/v1/` costs nothing.

2. **Use URL path versioning for public APIs**: It's the most visible and easiest for developers to understand.

3. **Share business logic**: Keep services and models version-agnostic. Only version the API layer.

4. **Support at most 2-3 active versions**: More than that becomes unmaintainable.

## Summary

API versioning prevents breaking changes from affecting existing clients. URL path versioning is the most common approach for public APIs. Share business logic between versions, limit active versions to 2-3, and only create new versions for breaking changes.

## Code Examples

**URL path versioning with separate URL configurations per version**

```python
# urls.py
from django.urls import path, include

urlpatterns = [
    path('api/v1/', include('myapp.api.v1.urls')),
    path('api/v2/', include('myapp.api.v2.urls')),
]
```


## Resources

- [Django URL Dispatcher](https://docs.djangoproject.com/en/6.0/topics/http/urls/) — Official URL routing documentation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*