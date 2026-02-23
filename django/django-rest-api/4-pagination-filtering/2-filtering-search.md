---
source_course: "django-rest-api"
source_lesson: "django-rest-api-filtering-search"
---

# Filtering and Search

## Introduction

APIs that return everything and leave filtering to the client waste bandwidth and processing power. Server-side filtering and search let clients request exactly the data they need, making your API efficient and user-friendly.

Allow clients to filter and search your API data efficiently.


## Key Concepts

**Filtering**: Narrowing results based on field values (e.g., `?category=tech&author=john`).

**Search**: Finding records that match a text query across multiple fields.

**Q Objects**: Django's tool for building complex queries with AND/OR logic.

**Full-Text Search**: PostgreSQL's built-in search with ranking, stemming, and relevance scoring.


## Real World Context

Every data-driven API needs filtering:
- **E-commerce**: Filter products by category, price range, brand, rating
- **Job boards**: Filter by location, salary, experience level
- **Analytics dashboards**: Filter metrics by date range, region, segment
- **Content platforms**: Search articles by keyword, author, or tag

## Basic Filtering

Filters are applied conditionally based on which query parameters are present, chaining `.filter()` calls to narrow the queryset:

```python
def api_articles(request):
    articles = Article.objects.all()
    
    # Filter by category
    category = request.GET.get('category')
    if category:
        articles = articles.filter(category__slug=category)
    
    # Filter by author
    author = request.GET.get('author')
    if author:
        articles = articles.filter(author__username=author)
    
    # Filter by published status
    is_published = request.GET.get('is_published')
    if is_published is not None:
        articles = articles.filter(is_published=is_published.lower() == 'true')
    
    # Filter by date range
    date_from = request.GET.get('date_from')
    date_to = request.GET.get('date_to')
    if date_from:
        articles = articles.filter(created_at__gte=date_from)
    if date_to:
        articles = articles.filter(created_at__lte=date_to)
    
    return JsonResponse({
        'results': list(articles.values('id', 'title', 'created_at'))
    })
```

Each filter is independent and optional. When no parameters are provided, the view returns all articles unfiltered.

## Search Implementation

Django's `Q` objects let you combine conditions with OR logic, enabling search across multiple fields with a single query:

```python
from django.db.models import Q

def api_articles(request):
    articles = Article.objects.all()
    
    # Search across multiple fields
    search = request.GET.get('search', '').strip()
    if search:
        articles = articles.filter(
            Q(title__icontains=search) |
            Q(body__icontains=search) |
            Q(author__username__icontains=search) |
            Q(tags__name__icontains=search)
        ).distinct()
    
    return JsonResponse({
        'results': list(articles.values('id', 'title')[:50])
    })
```

The `.distinct()` call is important when searching across related fields (like tags) to prevent duplicate results from JOINs.

## Full-Text Search (PostgreSQL)

PostgreSQL's built-in full-text search supports stemming, ranking by relevance, and weighted fields -- far more powerful than `icontains` for text-heavy applications:

```python
from django.contrib.postgres.search import (
    SearchVector, SearchQuery, SearchRank
)

def api_search_articles(request):
    query = request.GET.get('q', '')
    
    if not query:
        return JsonResponse({'results': []})
    
    # Create search vector
    vector = SearchVector('title', weight='A') + \
             SearchVector('body', weight='B') + \
             SearchVector('author__username', weight='C')
    
    search_query = SearchQuery(query)
    
    articles = Article.objects.annotate(
        search=vector,
        rank=SearchRank(vector, search_query)
    ).filter(
        search=search_query
    ).order_by('-rank')[:20]
    
    return JsonResponse({
        'query': query,
        'results': [{
            'id': a.id,
            'title': a.title,
            'relevance': a.rank
        } for a in articles]
    })
```

The `weight` parameter (`'A'`, `'B'`, `'C'`) controls how much each field contributes to the relevance score. Title matches rank higher than body matches.

## Filter Class

Encapsulating filter logic in a dedicated class keeps views clean and makes filters reusable across multiple endpoints:

```python
class ArticleFilter:
    """Reusable filter class."""
    
    def __init__(self, request):
        self.params = request.GET
    
    def apply(self, queryset):
        # Category filter
        category = self.params.get('category')
        if category:
            queryset = queryset.filter(category__slug=category)
        
        # Author filter
        author = self.params.get('author')
        if author:
            queryset = queryset.filter(author__username=author)
        
        # Published filter
        published = self.params.get('published')
        if published == 'true':
            queryset = queryset.filter(is_published=True)
        elif published == 'false':
            queryset = queryset.filter(is_published=False)
        
        # Date range
        created_after = self.params.get('created_after')
        created_before = self.params.get('created_before')
        if created_after:
            queryset = queryset.filter(created_at__gte=created_after)
        if created_before:
            queryset = queryset.filter(created_at__lte=created_before)
        
        # Search
        search = self.params.get('search', '').strip()
        if search:
            queryset = queryset.filter(
                Q(title__icontains=search) |
                Q(body__icontains=search)
            )
        
        return queryset


# Usage
def api_articles(request):
    articles = Article.objects.all()
    
    # Apply filters
    filter = ArticleFilter(request)
    articles = filter.apply(articles)
    
    # Apply ordering
    ordering = request.GET.get('ordering', '-created_at')
    if ordering.lstrip('-') in ['created_at', 'title', 'views']:
        articles = articles.order_by(ordering)
    
    # Paginate and return
    paginator = Paginator(request)
    result = paginator.paginate(articles)
    
    return JsonResponse({
        'results': [serialize_article(a) for a in result['items']],
        'pagination': result['meta']
    })
```

The view composes filtering, ordering, and pagination into a clean pipeline: build the queryset, apply filters, sort, then paginate.

## API Usage Examples

```
GET /api/articles/
GET /api/articles/?page=2&page_size=10
GET /api/articles/?category=tech&author=john
GET /api/articles/?search=django+tutorial
GET /api/articles/?created_after=2024-01-01&ordering=-views
GET /api/articles/?published=true&category=python&page=1
```

## Common Pitfalls

1. **SQL injection via filter fields**: Never pass user input directly to `filter()` kwargs. Whitelist allowed filter fields.

2. **Missing indexes**: Filtering on non-indexed fields causes full table scans. Add database indexes for frequently filtered columns.

3. **Case sensitivity**: `filter(title=search)` is case-sensitive in PostgreSQL. Use `icontains` or `Lower()` for case-insensitive filtering.

## Best Practices

1. **Whitelist filter fields**: Only allow filtering on safe, indexed fields.

2. **Combine with pagination**: Always paginate filtered results.

3. **Use Q objects for OR logic**: Regular `filter()` chains use AND. Use `Q()` with `|` for OR queries.

4. **Consider full-text search**: For text-heavy applications, PostgreSQL's full-text search is faster and more relevant than `icontains`.

## Summary

Server-side filtering and search make APIs efficient by returning only relevant data. Use query parameters for filtering, Q objects for complex OR queries, and PostgreSQL's full-text search for relevance-ranked results. Always whitelist filter fields and add database indexes for performance.

## Code Examples

**Filtering and searching across multiple fields with Q objects**

```python
from django.db.models import Q
from django.http import JsonResponse

def api_articles(request):
    articles = Article.objects.all()
    category = request.GET.get('category')
    if category:
        articles = articles.filter(category__slug=category)
    search = request.GET.get('search', '').strip()
    if search:
        articles = articles.filter(
            Q(title__icontains=search) |
            Q(body__icontains=search)
        ).distinct()
    return JsonResponse({'results': list(articles.values('id', 'title')[:50])})
```


## Resources

- [Full Text Search](https://docs.djangoproject.com/en/6.0/ref/contrib/postgres/search/) — PostgreSQL full-text search in Django

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*