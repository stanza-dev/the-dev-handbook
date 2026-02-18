---
source_course: "django-rest-api"
source_lesson: "django-rest-api-versioning-strategies"
---

# API Versioning Strategies

Versioning prevents breaking changes from affecting existing clients. Choose a strategy that fits your needs.

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

## Query Parameter Versioning

```python
# Simple but less RESTful
def api_articles(request):
    version = request.GET.get('version', 'v1')
    # ...
```

## Version-Aware Views

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

## Deprecation Strategy

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

## Resources

- [Django URL Dispatcher](https://docs.djangoproject.com/en/6.0/topics/http/urls/) — Official URL routing documentation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*