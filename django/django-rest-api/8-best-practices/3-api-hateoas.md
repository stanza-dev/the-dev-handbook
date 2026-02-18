---
source_course: "django-rest-api"
source_lesson: "django-rest-api-hateoas"
---

# HATEOAS and Hypermedia

## Introduction

HATEOAS (Hypermedia as the Engine of Application State) is a REST constraint where responses include links to related resources and available actions. It makes APIs self-documenting and discoverable.

## Key Concepts

**Hypermedia**: Links embedded in responses that tell clients what they can do next.

**Self-Describing API**: Clients discover capabilities from responses, not hardcoded URLs.

**Link Relations**: Standard names describing what each link represents (self, next, author, etc.).

## Deep Dive

### Adding Links to Responses

```python
def get_article(request, pk):
    article = Article.objects.get(pk=pk)
    
    return JsonResponse({
        'data': {
            'id': article.id,
            'title': article.title,
            'body': article.body,
        },
        'links': {
            'self': f'/api/articles/{pk}/',
            'author': f'/api/users/{article.author_id}/',
            'comments': f'/api/articles/{pk}/comments/',
            'category': f'/api/categories/{article.category_id}/',
        },
        'actions': {
            'edit': f'/api/articles/{pk}/' if can_edit(request.user, article) else None,
            'delete': f'/api/articles/{pk}/' if can_delete(request.user, article) else None,
            'publish': f'/api/articles/{pk}/publish/' if not article.is_published else None,
        }
    })
```

### Pagination Links

```python
def list_articles(request):
    page = int(request.GET.get('page', 1))
    base_url = '/api/articles/'
    
    return JsonResponse({
        'data': articles_data,
        'links': {
            'self': f'{base_url}?page={page}',
            'first': f'{base_url}?page=1',
            'last': f'{base_url}?page={total_pages}',
            'prev': f'{base_url}?page={page-1}' if page > 1 else None,
            'next': f'{base_url}?page={page+1}' if page < total_pages else None,
        }
    })
```

### API Root Discovery

```python
def api_root(request):
    """Entry point for API discovery."""
    return JsonResponse({
        'links': {
            'articles': '/api/articles/',
            'users': '/api/users/',
            'categories': '/api/categories/',
            'docs': '/api/docs/',
            'schema': '/api/schema/',
        },
        'version': 'v1',
    })
```

## Best Practices

1. **Include self link**: Every resource should link to itself.
2. **Use absolute URLs**: Easier for clients to use directly.
3. **Conditional actions**: Only show links/actions the user can perform.
4. **Standard link relations**: Use established names (self, next, prev, author).

## Summary

HATEOAS makes APIs discoverable by embedding links to related resources and available actions. Include self-links, pagination links, and conditional action links based on user permissions.

## Resources

- [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html) — Steps toward REST maturity including HATEOAS

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*