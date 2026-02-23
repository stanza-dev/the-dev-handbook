---
source_course: "django-rest-api"
source_lesson: "django-rest-api-pagination"
---

# Implementing Pagination

## Introduction

Returning thousands of records in a single response is a recipe for slow APIs, crashed clients, and angry users. Pagination breaks large datasets into manageable chunks, improving performance for both server and client.

Pagination is essential for APIs returning large datasets. It improves performance and user experience.


## Key Concepts

**Page-Based Pagination**: The most common approach, using `?page=2&per_page=20` to navigate through numbered pages.

**Cursor-Based Pagination**: Uses an opaque token to bookmark position, ideal for real-time feeds and infinite scrolling.

**Offset-Based Pagination**: Uses `?offset=40&limit=20` for fine-grained control over result position.

**Django Paginator**: Django's built-in `Paginator` class that handles page calculation and boundary checking.


## Real World Context

Pagination is essential for:
- **Social media feeds**: Infinite scroll loading posts in batches
- **E-commerce catalogs**: Browsing products page by page
- **Admin dashboards**: Managing thousands of records without overwhelming the browser
- **Search results**: Displaying results in pages with navigation controls

## Page-Based Pagination

The most common approach:

```python
from django.http import JsonResponse
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger


def api_articles(request):
    # Get pagination parameters
    page = request.GET.get('page', 1)
    per_page = min(int(request.GET.get('per_page', 20)), 100)  # Max 100
    
    # Get queryset
    articles = Article.objects.all().order_by('-created_at')
    
    # Paginate
    paginator = Paginator(articles, per_page)
    
    try:
        page_obj = paginator.page(page)
    except PageNotAnInteger:
        page_obj = paginator.page(1)
    except EmptyPage:
        page_obj = paginator.page(paginator.num_pages)
    
    # Serialize
    articles_data = [{
        'id': a.id,
        'title': a.title,
        'created_at': a.created_at.isoformat()
    } for a in page_obj]
    
    return JsonResponse({
        'results': articles_data,
        'pagination': {
            'page': page_obj.number,
            'per_page': per_page,
            'total_pages': paginator.num_pages,
            'total_count': paginator.count,
            'has_next': page_obj.has_next(),
            'has_previous': page_obj.has_previous(),
        }
    })
```

Response:
```json
{
  "results": [...],
  "pagination": {
    "page": 2,
    "per_page": 20,
    "total_pages": 15,
    "total_count": 287,
    "has_next": true,
    "has_previous": true
  }
}
```

The pagination metadata gives clients everything they need to build navigation controls: current position, total pages, and next/previous flags.

## Offset-Based Pagination

Offset-based pagination uses `?offset=40&limit=20` instead of page numbers, giving clients fine-grained control over the starting position:

```python
def api_articles(request):
    offset = int(request.GET.get('offset', 0))
    limit = min(int(request.GET.get('limit', 20)), 100)
    
    articles = Article.objects.all().order_by('-created_at')
    total = articles.count()
    
    articles_data = [{
        'id': a.id,
        'title': a.title
    } for a in articles[offset:offset + limit]]
    
    return JsonResponse({
        'results': articles_data,
        'offset': offset,
        'limit': limit,
        'total': total,
        'has_more': offset + limit < total
    })
```

Offset pagination is simple but can skip or duplicate items when data changes between requests. For real-time feeds, consider cursor-based pagination instead.

## Cursor-Based Pagination

Better for real-time data and infinite scrolling:

```python
import base64
from django.http import JsonResponse


def encode_cursor(pk, created_at):
    """Encode cursor as base64."""
    cursor = f"{pk}:{created_at.isoformat()}"
    return base64.b64encode(cursor.encode()).decode()

def decode_cursor(cursor):
    """Decode base64 cursor."""
    decoded = base64.b64decode(cursor.encode()).decode()
    pk, created_at = decoded.split(':')
    return int(pk), created_at


def api_articles(request):
    limit = min(int(request.GET.get('limit', 20)), 100)
    cursor = request.GET.get('cursor')
    
    articles = Article.objects.all().order_by('-created_at', '-id')
    
    if cursor:
        pk, created_at = decode_cursor(cursor)
        articles = articles.filter(
            created_at__lte=created_at
        ).exclude(
            created_at=created_at,
            id__gte=pk
        )
    
    articles = list(articles[:limit + 1])  # Get one extra to check for more
    has_more = len(articles) > limit
    articles = articles[:limit]
    
    articles_data = [{
        'id': a.id,
        'title': a.title,
        'created_at': a.created_at.isoformat()
    } for a in articles]
    
    response = {
        'results': articles_data,
        'has_more': has_more
    }
    
    if articles and has_more:
        last = articles[-1]
        response['next_cursor'] = encode_cursor(last.id, last.created_at)
    
    return JsonResponse(response)
```

The cursor is a base64-encoded string containing the last item's ID and timestamp. Fetching one extra record (`limit + 1`) lets us determine if more pages exist without a separate COUNT query.

## Pagination Class

A reusable pagination class encapsulates parameter parsing and page calculation, keeping your views clean:

```python
class Paginator:
    """Reusable pagination class."""
    
    def __init__(self, request, default_page_size=20, max_page_size=100):
        self.page = int(request.GET.get('page', 1))
        self.page_size = min(
            int(request.GET.get('page_size', default_page_size)),
            max_page_size
        )
    
    def paginate(self, queryset):
        from django.core.paginator import Paginator as DjangoPaginator
        
        paginator = DjangoPaginator(queryset, self.page_size)
        page_obj = paginator.get_page(self.page)
        
        return {
            'items': list(page_obj),
            'meta': {
                'page': page_obj.number,
                'page_size': self.page_size,
                'total_pages': paginator.num_pages,
                'total_count': paginator.count,
            }
        }


# Usage
def api_articles(request):
    paginator = Paginator(request)
    articles = Article.objects.all()
    result = paginator.paginate(articles)
    
    return JsonResponse({
        'results': [serialize_article(a) for a in result['items']],
        'pagination': result['meta']
    })
```

With this helper class, adding pagination to any view is a two-line operation: create the paginator and call `paginate()` on your queryset.

## Common Pitfalls

1. **No maximum page size**: Allowing `?per_page=1000000` lets clients crash your server. Always cap the maximum.

2. **COUNT queries on large tables**: `paginator.count` runs `SELECT COUNT(*)` which can be slow on millions of rows. Consider estimating or caching.

3. **Offset pagination on changing data**: If records are added/removed between page requests, items can be skipped or duplicated. Use cursor pagination for real-time data.

## Best Practices

1. **Cap page sizes**: Set a maximum (e.g., 100) and enforce it server-side.

2. **Include pagination metadata**: Return total count, current page, total pages, and has_next/has_previous flags.

3. **Use cursor pagination for real-time data**: It handles insertions and deletions gracefully.

4. **Default to sensible page sizes**: 20-25 items per page is a good starting point.

## Summary

Pagination prevents performance issues by breaking large datasets into pages. Use Django's built-in Paginator for page-based pagination, implement cursor-based pagination for real-time feeds, and always cap maximum page sizes. Include pagination metadata in responses so clients can navigate efficiently.

## Code Examples

**Page-based pagination using Django's built-in Paginator**

```python
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from django.http import JsonResponse

def api_articles(request):
    page = request.GET.get('page', 1)
    per_page = min(int(request.GET.get('per_page', 20)), 100)
    articles = Article.objects.all().order_by('-created_at')
    paginator = Paginator(articles, per_page)
    try:
        page_obj = paginator.page(page)
    except (PageNotAnInteger, EmptyPage):
        page_obj = paginator.page(1)
    return JsonResponse({
        'results': [{'id': a.id, 'title': a.title} for a in page_obj],
        'pagination': {
            'page': page_obj.number,
            'total_pages': paginator.num_pages,
            'total_count': paginator.count,
        }
    })
```


## Resources

- [Django Pagination](https://docs.djangoproject.com/en/6.0/topics/pagination/) — Official pagination documentation

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*