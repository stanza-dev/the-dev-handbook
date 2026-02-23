---
source_course: "django-rest-api"
source_lesson: "django-rest-api-api-design-principles"
---

# API Design Principles

## Introduction

Well-designed APIs are intuitive, consistent, and a pleasure to use. Bad APIs frustrate developers, increase support costs, and slow down integration. Following established design principles ensures your API is professional and maintainable.

Well-designed APIs are intuitive, consistent, and easy to use. Follow these principles for professional API development.


## Key Concepts

**RESTful Design**: Using HTTP methods, status codes, and resource-based URLs to create predictable APIs.

**Response Envelope**: A consistent wrapper structure for all API responses containing data, errors, and metadata.

**HATEOAS**: Including links in responses so clients can discover available actions without hardcoding URLs.

**Idempotency**: Ensuring that repeated identical requests produce the same result, enabling safe retries.


## Real World Context

API design principles matter for:
- **Developer adoption**: Intuitive APIs attract more users and reduce onboarding time
- **Long-term maintenance**: Consistent patterns make APIs easier to extend and maintain
- **Team collaboration**: Shared conventions reduce code review friction
- **Integration reliability**: Predictable behavior reduces integration bugs

## RESTful URL Design

Organize URLs around resources (nouns) and use HTTP methods for actions. Nest related resources logically, and limit nesting to two levels:

```python
# urls.py - Good URL patterns
urlpatterns = [
    # Resources as nouns
    path('api/articles/', ArticleListView.as_view()),
    path('api/articles/<int:pk>/', ArticleDetailView.as_view()),
    
    # Nested resources
    path('api/articles/<int:article_id>/comments/', CommentListView.as_view()),
    path('api/users/<int:user_id>/articles/', UserArticleListView.as_view()),
    
    # Actions as sub-resources (when needed)
    path('api/articles/<int:pk>/publish/', ArticlePublishView.as_view()),
    
    # Filtering via query parameters
    # GET /api/articles/?status=published&author=1&page=2
]
```

**URL Guidelines:**
- Use nouns, not verbs (GET /articles not GET /getArticles)
- Use plural nouns (/articles not /article)
- Use hyphens for multi-word resources (/user-profiles)
- Keep URLs lowercase
- Version your API (/api/v1/articles)

## Consistent Response Format

Define helper functions that enforce a standard envelope structure for all API responses, ensuring clients always know where to find data and errors:

```python
# utils.py
from django.http import JsonResponse


def api_response(data=None, message=None, status=200):
    """Create consistent API response."""
    response = {
        'success': 200 <= status < 300,
        'status': status,
    }
    
    if message:
        response['message'] = message
    
    if data is not None:
        response['data'] = data
    
    return JsonResponse(response, status=status)


def api_error(message, code=None, details=None, status=400):
    """Create consistent error response."""
    response = {
        'success': False,
        'status': status,
        'error': {
            'message': message,
        }
    }
    
    if code:
        response['error']['code'] = code
    
    if details:
        response['error']['details'] = details
    
    return JsonResponse(response, status=status)
```

```python
# views.py
def get_article(request, pk):
    try:
        article = Article.objects.get(pk=pk)
    except Article.DoesNotExist:
        return api_error(
            message='Article not found',
            code='ARTICLE_NOT_FOUND',
            status=404
        )
    
    return api_response(data={
        'id': article.id,
        'title': article.title,
        'body': article.body,
        'created_at': article.created_at.isoformat(),
    })
```

Using `api_response()` and `api_error()` across all endpoints ensures that success and error payloads always follow the same structure.

## Use Appropriate HTTP Methods

Each HTTP method carries specific semantics. Map them to your view methods with clear docstrings describing their behavior:

```python
# Each method has specific semantics
class ArticleView(View):
    def get(self, request, pk=None):
        """GET - Retrieve resource (safe, idempotent)."""
        if pk:
            return self.get_detail(pk)
        return self.get_list()
    
    def post(self, request):
        """POST - Create resource (not idempotent)."""
        return self.create_article(request)
    
    def put(self, request, pk):
        """PUT - Replace entire resource (idempotent)."""
        return self.replace_article(pk, request)
    
    def patch(self, request, pk):
        """PATCH - Partial update (not guaranteed idempotent)."""
        return self.update_article(pk, request)
    
    def delete(self, request, pk):
        """DELETE - Remove resource (idempotent)."""
        return self.delete_article(pk)
```

GET and DELETE are idempotent (safe to retry), while POST is not. PATCH is not guaranteed idempotent per the HTTP spec.

## Use Correct Status Codes

Return the most specific status code for each outcome. Clients rely on these codes for control flow, not just display:

```python
from django.http import HttpResponse, JsonResponse

# Success codes
return JsonResponse(data, status=200)  # OK - Successful GET
return JsonResponse(data, status=201)  # Created - Successful POST
return HttpResponse(status=204)         # No Content - Successful DELETE

# Client error codes
return JsonResponse(error, status=400)  # Bad Request - Validation error
return JsonResponse(error, status=401)  # Unauthorized - Not authenticated
return JsonResponse(error, status=403)  # Forbidden - Not authorized
return JsonResponse(error, status=404)  # Not Found
return JsonResponse(error, status=409)  # Conflict - Duplicate resource
return JsonResponse(error, status=422)  # Unprocessable Entity - Semantic error

# Server error codes
return JsonResponse(error, status=500)  # Internal Server Error
return JsonResponse(error, status=503)  # Service Unavailable
```

The distinction between 400 (malformed request), 422 (valid syntax, invalid semantics), and 409 (resource conflict) helps clients handle each error type differently.

## HATEOAS (Hypermedia Links)

Include links to related resources and available actions so clients can discover the API's capabilities from responses:

```python
def get_article(request, pk):
    article = Article.objects.get(pk=pk)
    
    return api_response(data={
        'id': article.id,
        'title': article.title,
        # Include related resource links
        '_links': {
            'self': request.build_absolute_uri(),
            'author': f'/api/users/{article.author_id}/',
            'comments': f'/api/articles/{pk}/comments/',
            'edit': f'/api/articles/{pk}/' if can_edit(request.user, article) else None,
        }
    })
```

The `actions` dict conditionally includes links based on the current user's permissions. A `None` value signals that the action is not available.

## Pagination Best Practices

Always include pagination metadata so clients can build navigation controls and know the full extent of the dataset:

```python
def get_articles(request):
    page = int(request.GET.get('page', 1))
    per_page = min(int(request.GET.get('per_page', 20)), 100)  # Max 100
    
    articles = Article.objects.filter(status='published')
    paginator = Paginator(articles, per_page)
    page_obj = paginator.get_page(page)
    
    return api_response(data={
        'items': list(page_obj.object_list.values()),
        'pagination': {
            'page': page,
            'per_page': per_page,
            'total_pages': paginator.num_pages,
            'total_items': paginator.count,
            'has_next': page_obj.has_next(),
            'has_previous': page_obj.has_previous(),
        }
    })
```

The `min()` call caps `per_page` at 100 to prevent clients from requesting the entire dataset in one call. The pagination metadata includes everything clients need for both numbered and infinite-scroll UIs.

## Common Pitfalls

1. **Inconsistent naming**: Mixing camelCase and snake_case, or singular and plural nouns, confuses developers.

2. **Using verbs in URLs**: `/api/getArticles` should be `GET /api/articles/`. Let HTTP methods express the action.

3. **Returning 200 for errors**: Always use appropriate status codes (400, 401, 403, 404, 422, 500).

## Best Practices

1. **Use plural nouns for resources**: `/articles/` not `/article/`.

2. **Return appropriate status codes**: Use 201 for creation, 204 for deletion, 400 for validation errors.

3. **Version your API from day one**: Start with `/api/v1/` even if v2 seems unlikely.

4. **Use consistent response envelopes**: Same structure for success and error responses across all endpoints.

## Summary

Professional API design follows RESTful conventions: resource-based URLs with plural nouns, appropriate HTTP methods and status codes, consistent response formats, and versioning from the start. These principles make your API intuitive, maintainable, and enjoyable for developers to use.

## Code Examples

**Consistent API response helper functions for success and error responses**

```python
from django.http import JsonResponse

def api_response(data=None, message=None, status=200):
    response = {'success': 200 <= status < 300, 'status': status}
    if message:
        response['message'] = message
    if data is not None:
        response['data'] = data
    return JsonResponse(response, status=status)

def api_error(message, code=None, details=None, status=400):
    response = {'success': False, 'status': status,
                'error': {'message': message}}
    if code:
        response['error']['code'] = code
    if details:
        response['error']['details'] = details
    return JsonResponse(response, status=status)
```


## Resources

- [REST API Design](https://restfulapi.net/) — REST API design guidelines

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*