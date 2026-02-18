---
source_course: "django-rest-api"
source_lesson: "django-rest-api-ordering"
---

# Ordering and Sorting

## Introduction

Users expect to sort data—newest first, alphabetically, by popularity. A well-designed API lets clients control ordering while protecting against performance issues and injection attacks.

## Key Concepts

**Ordering**: Arranging results based on one or more fields in ascending or descending order.

**Multi-field Sorting**: Sorting by multiple criteria (e.g., category first, then date).

**Sort Direction**: Ascending (A-Z, oldest-newest) or descending (Z-A, newest-oldest), often indicated by a `-` prefix.

## Real World Context

Every list endpoint needs sorting:
- E-commerce: Sort by price, rating, newest
- Social: Sort by recent, popular, trending
- Admin: Sort by any column for data management

## Deep Dive

### Basic Ordering

```python
def api_articles(request):
    # GET /api/articles/?ordering=-created_at
    ordering = request.GET.get('ordering', '-created_at')
    
    # Whitelist allowed fields
    allowed = ['created_at', '-created_at', 'title', '-title', 'views', '-views']
    
    if ordering not in allowed:
        ordering = '-created_at'  # Default
    
    articles = Article.objects.all().order_by(ordering)
    return JsonResponse({'data': list(articles.values('id', 'title')[:20])})
```

### Multi-field Ordering

```python
def api_articles(request):
    # GET /api/articles/?ordering=category,-created_at
    ordering_param = request.GET.get('ordering', '-created_at')
    
    allowed_fields = {'created_at', 'title', 'views', 'category'}
    order_fields = []
    
    for field in ordering_param.split(','):
        field_name = field.lstrip('-')
        if field_name in allowed_fields:
            order_fields.append(field)
    
    if not order_fields:
        order_fields = ['-created_at']
    
    articles = Article.objects.all().order_by(*order_fields)
    return JsonResponse({'data': list(articles.values()[:20])})
```

### Reusable Ordering Mixin

```python
class OrderingMixin:
    ordering_fields = []
    default_ordering = '-created_at'
    
    def get_ordering(self, request):
        param = request.GET.get('ordering', self.default_ordering)
        fields = []
        
        for field in param.split(','):
            field_name = field.lstrip('-')
            if field_name in self.ordering_fields:
                fields.append(field)
        
        return fields or [self.default_ordering]
    
    def apply_ordering(self, request, queryset):
        return queryset.order_by(*self.get_ordering(request))

class ArticleAPIView(OrderingMixin, View):
    ordering_fields = ['created_at', 'title', 'views']
    default_ordering = '-created_at'
    
    def get(self, request):
        qs = self.apply_ordering(request, Article.objects.all())
        return JsonResponse({'data': list(qs.values()[:20])})
```

## Common Pitfalls

1. **Allowing arbitrary fields**: Never use `order_by(request.GET.get('sort'))` directly. Always whitelist allowed fields.

2. **Forgetting indexes**: Sorting on non-indexed fields causes slow queries on large tables.

3. **Case-sensitive sorting**: `order_by('title')` is case-sensitive in PostgreSQL. Use `Lower()` for case-insensitive.

## Best Practices

1. **Whitelist allowed fields**: Only allow sorting on indexed, safe fields.
2. **Provide sensible defaults**: Most recent first is usually expected.
3. **Document available options**: Let clients know which fields support sorting.
4. **Add database indexes**: Index columns that are frequently sorted.

## Summary

Ordering lets clients control result order. Always whitelist allowed sort fields, support both ascending and descending with `-` prefix, and ensure sorted columns are indexed for performance.

## Resources

- [QuerySet order_by](https://docs.djangoproject.com/en/6.0/ref/models/querysets/#order-by) — Django QuerySet ordering

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*