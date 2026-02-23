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

## Real World Context

HATEOAS is used in production by:
- **GitHub API**: Responses include URLs for related resources and pagination
- **PayPal API**: Payment responses include links for capture, void, and refund actions
- **AWS APIs**: Resource responses include ARNs and links to related services
- **HAL-based APIs**: A popular format for hypermedia-driven REST APIs

## Deep Dive

### Adding Links to Responses

Include a `links` dict for navigation to related resources and an `actions` dict for operations the current user can perform:

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

Conditional action links (returning `None` when unavailable) tell clients what the user can do without requiring a separate permissions endpoint.

### Pagination Links

Pagination links let clients navigate through pages by following URLs instead of constructing them manually:

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

Setting `prev` and `next` to `None` at the boundaries signals to clients that there are no more pages in that direction.

### API Root Discovery

A root endpoint serves as the entry point for the entire API, listing all available top-level resources:

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

Clients bookmark this single URL and discover everything else by following links. When URLs change, only the root response needs to be updated.

## Common Pitfalls

1. **Hardcoded URLs in clients**: If clients build URLs from patterns instead of following links, URL changes break them.

2. **Including actions the user cannot perform**: Only include links for actions the authenticated user is authorized to do.

3. **Over-engineering**: Not every API needs full HATEOAS. Start with self-links and pagination, add more as needed.

## Best Practices

1. **Include self link**: Every resource should link to itself.
2. **Use absolute URLs**: Easier for clients to use directly.
3. **Conditional actions**: Only show links/actions the user can perform.
4. **Standard link relations**: Use established names (self, next, prev, author).

## Summary

HATEOAS makes APIs discoverable by embedding links to related resources and available actions. Include self-links, pagination links, and conditional action links based on user permissions.

## Code Examples

**HATEOAS response with resource links and conditional actions**

```python
def get_article(request, pk):
    article = Article.objects.get(pk=pk)
    return JsonResponse({
        'data': {
            'id': article.id,
            'title': article.title,
        },
        'links': {
            'self': f'/api/articles/{pk}/',
            'author': f'/api/users/{article.author_id}/',
            'comments': f'/api/articles/{pk}/comments/',
        },
        'actions': {
            'edit': f'/api/articles/{pk}/' if can_edit(request.user, article) else None,
        }
    })
```


## Resources

- [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html) — Steps toward REST maturity including HATEOAS

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*