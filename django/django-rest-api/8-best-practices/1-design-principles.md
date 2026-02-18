---
source_course: "django-rest-api"
source_lesson: "django-rest-api-api-design-principles"
---

# API Design Principles

Well-designed APIs are intuitive, consistent, and easy to use. Follow these principles for professional API development.

## RESTful URL Design

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

## Use Appropriate HTTP Methods

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
        """PATCH - Partial update (idempotent)."""
        return self.update_article(pk, request)
    
    def delete(self, request, pk):
        """DELETE - Remove resource (idempotent)."""
        return self.delete_article(pk)
```

## Use Correct Status Codes

```python
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

## HATEOAS (Hypermedia Links)

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

## Pagination Best Practices

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

## Resources

- [REST API Design](https://restfulapi.net/) — REST API design guidelines

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*