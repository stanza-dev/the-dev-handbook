---
source_course: "django-rest-api"
source_lesson: "django-rest-api-url-routing"
---

# URL Routing for APIs

## Introduction

URL design is one of the most visible aspects of your API. Well-designed URLs are intuitive, consistent, and self-documenting. They tell developers exactly what resource they're working with before they even read the documentation.

## Key Concepts

**URL Pattern**: A route definition that maps a URL path to a view function. Django uses `path()` and `re_path()` for pattern matching.

**Path Converters**: Built-in converters like `<int:pk>` that capture and convert URL segments to Python types.

**URL Namespacing**: Organizing URLs into logical groups with prefixes like `/api/v1/` to avoid conflicts and enable versioning.

**Trailing Slashes**: Django's convention of using trailing slashes (`/articles/`) and the `APPEND_SLASH` setting.

## Real World Context

Your URL structure directly impacts:
- **Developer experience**: Intuitive URLs reduce onboarding time
- **API versioning**: Clean paths like `/api/v1/` vs `/api/v2/` enable smooth migrations
- **SEO and caching**: Consistent URL patterns improve cacheability
- **Documentation**: Good URLs are self-documenting

## Deep Dive

### Basic API URL Configuration

Use `include()` to keep your API URLs in a separate file, then mount them under a prefix like `api/` in the project-level configuration:

```python
# myproject/urls.py
from django.urls import path, include

urlpatterns = [
    path('api/', include('myapp.api_urls')),
]
```

```python
# myapp/api_urls.py
from django.urls import path
from . import views

urlpatterns = [
    # Collection endpoints
    path('articles/', views.article_list, name='api-article-list'),
    path('users/', views.user_list, name='api-user-list'),
    
    # Detail endpoints with path converters
    path('articles/<int:pk>/', views.article_detail, name='api-article-detail'),
    path('users/<int:pk>/', views.user_detail, name='api-user-detail'),
    
    # Nested resources
    path('articles/<int:article_id>/comments/', views.article_comments, name='api-article-comments'),
    
    # Custom actions
    path('articles/<int:pk>/publish/', views.article_publish, name='api-article-publish'),
]
```

Naming each URL pattern (e.g., `name='api-article-list'`) enables reverse lookups with `reverse()`, making your code resilient to URL changes.

### Path Converters

Django provides several built-in path converters that capture URL segments and automatically convert them to the correct Python type:

```python
# Built-in converters
path('articles/<int:pk>/', ...)      # Integer: /articles/42/
path('articles/<slug:slug>/', ...)   # Slug: /articles/my-post/
path('files/<path:filepath>/', ...)  # Path with slashes: /files/docs/readme.txt
path('users/<uuid:id>/', ...)        # UUID: /users/550e8400-e29b-41d4.../
path('tags/<str:name>/', ...)        # String (default): /tags/python/
```

The `<int:pk>` converter rejects non-integer values with a 404 automatically, saving you from manual type-checking in the view.

### API Versioning with URLs

Separate versioned URL files let you evolve your API without breaking existing clients. Each version gets its own URL module and namespace:

```python
# myproject/urls.py
urlpatterns = [
    path('api/v1/', include('myapp.api.v1.urls')),
    path('api/v2/', include('myapp.api.v2.urls')),
]
```

```python
# myapp/api/v1/urls.py
from django.urls import path
from . import views

app_name = 'api-v1'  # Namespace

urlpatterns = [
    path('articles/', views.ArticleListView.as_view(), name='articles'),
]
```

The `app_name` attribute creates a namespace so you can reference URLs as `api-v1:articles` without colliding with other apps.

### Using URL Names in Code

The `reverse()` function generates URLs from pattern names, so your code stays correct even when URL paths change:

```python
from django.urls import reverse

# Generate URL from name
url = reverse('api-article-detail', kwargs={'pk': 42})
# Result: '/api/articles/42/'

# In responses (HATEOAS)
def article_detail(request, pk):
    article = get_object_or_404(Article, pk=pk)
    return JsonResponse({
        'id': article.id,
        'title': article.title,
        'links': {
            'self': request.build_absolute_uri(),
            'comments': reverse('api-article-comments', kwargs={'article_id': pk}),
        }
    })
```

Using `reverse()` instead of hardcoded strings ensures your links stay valid as URL patterns evolve. The `request.build_absolute_uri()` helper generates a full URL including the domain.

### RESTful URL Patterns

REST conventions dictate using plural nouns for resource collections and relying on HTTP methods for actions, not verb-based URLs:

```python
# Following REST conventions
urlpatterns = [
    # Resource collections (plural nouns)
    path('articles/', views.articles, name='articles'),      # GET=list, POST=create
    path('articles/<int:pk>/', views.article, name='article'), # GET, PUT, PATCH, DELETE
    
    # Nested resources
    path('articles/<int:pk>/comments/', views.comments),
    path('users/<int:pk>/articles/', views.user_articles),
    
    # Avoid: action-based URLs
    # path('createArticle/', ...)  # BAD
    # path('getArticles/', ...)    # BAD
]
```

The commented-out `BAD` examples show the anti-pattern of embedding actions in URLs. Instead, let `GET /articles/` handle listing and `POST /articles/` handle creation.

## Common Pitfalls

1. **Using verbs in URLs**: `/api/getArticles/` should be `GET /api/articles/`. Let HTTP methods express the action.

2. **Inconsistent pluralization**: Mixing `/article/` and `/users/`. Pick one convention (plural is standard) and stick with it.

3. **Deep nesting**: `/users/1/posts/5/comments/3/likes/` is hard to work with. Limit nesting to 2 levels max.

## Best Practices

1. **Use plural nouns**: `/articles/` not `/article/`
2. **Use hyphens for multi-word**: `/user-profiles/` not `/userProfiles/`
3. **Keep URLs lowercase**: `/api/articles/` not `/API/Articles/`
4. **Version from the start**: `/api/v1/` even if you don't plan v2 yet
5. **Name all URL patterns**: Enables `reverse()` and easier maintenance
6. **Use namespaces**: `app_name = 'api'` prevents naming collisions

## Summary

Django's URL routing maps HTTP requests to view functions using `path()` patterns. Design RESTful URLs using plural nouns for resources, path converters for dynamic segments, and namespaces for organization. Always name your URL patterns to enable reverse URL lookups. Keep URLs consistent, lowercase, and limit nesting depth for maintainable APIs.

## Code Examples

**API URL routing with path converters and namespacing**

```python
from django.urls import path, include

# myapp/api_urls.py
urlpatterns = [
    path('articles/', views.article_list, name='api-article-list'),
    path('articles/<int:pk>/', views.article_detail, name='api-article-detail'),
    path('articles/<int:article_id>/comments/', views.article_comments),
]

# project/urls.py
urlpatterns = [
    path('api/v1/', include('myapp.api_urls')),
]
```


## Resources

- [URL Dispatcher](https://docs.djangoproject.com/en/6.0/topics/http/urls/) — Django URL routing documentation
- [Path Converters](https://docs.djangoproject.com/en/6.0/topics/http/urls/#path-converters) — Built-in path converters reference

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*