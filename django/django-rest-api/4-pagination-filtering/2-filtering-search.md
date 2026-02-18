---
source_course: "django-rest-api"
source_lesson: "django-rest-api-filtering-search"
---

# Filtering and Search

Allow clients to filter and search your API data efficiently.

## Basic Filtering

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

## Search Implementation

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

## Full-Text Search (PostgreSQL)

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

## Filter Class

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

## API Usage Examples

```
GET /api/articles/
GET /api/articles/?page=2&page_size=10
GET /api/articles/?category=tech&author=john
GET /api/articles/?search=django+tutorial
GET /api/articles/?created_after=2024-01-01&ordering=-views
GET /api/articles/?published=true&category=python&page=1
```

## Resources

- [Full Text Search](https://docs.djangoproject.com/en/6.0/ref/contrib/postgres/search/) — PostgreSQL full-text search in Django

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*