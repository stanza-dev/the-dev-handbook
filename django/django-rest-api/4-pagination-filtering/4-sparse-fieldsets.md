---
source_course: "django-rest-api"
source_lesson: "django-rest-api-sparse-fieldsets"
---

# Sparse Fieldsets

## Introduction

Not every client needs every field. Mobile apps might want minimal data to save bandwidth, while admin dashboards need everything. Sparse fieldsets let clients request only the fields they need.

## Key Concepts

**Sparse Fieldsets**: A technique where clients specify which fields to include in the response.

**Field Selection**: Using query parameters like `?fields=id,title,author` to limit response data.

**Over-fetching**: Returning more data than clients need, wasting bandwidth and processing.

## Real World Context

Sparse fieldsets are essential for:
- **Mobile apps**: Minimize data transfer on slow connections
- **List views**: Show only summary fields, not full content
- **Performance**: Reduce serialization overhead for unused fields
- **GraphQL alternative**: Provides similar benefits in REST APIs

## Deep Dive

### Basic Implementation

```python
def api_articles(request):
    # GET /api/articles/?fields=id,title,author
    requested = request.GET.get('fields', '').split(',')
    requested = [f.strip() for f in requested if f.strip()]
    
    # Define all available fields
    all_fields = ['id', 'title', 'body', 'author', 'created_at', 'views']
    
    # Filter to allowed fields (or use all if none specified)
    fields = [f for f in requested if f in all_fields] or all_fields
    
    articles = Article.objects.all()[:20]
    
    data = []
    for article in articles:
        item = {}
        for field in fields:
            if field == 'author':
                item[field] = article.author.username
            else:
                item[field] = getattr(article, field, None)
        data.append(item)
    
    return JsonResponse({'data': data, 'fields': fields})
```

### With Serializer

```python
class ArticleSerializer:
    all_fields = ['id', 'title', 'body', 'author', 'created_at', 'views']
    
    def __init__(self, instance, fields=None):
        self.instance = instance
        self.fields = fields or self.all_fields
    
    def serialize_one(self, obj):
        data = {}
        for field in self.fields:
            if field in self.all_fields:
                value = getattr(obj, field, None)
                if hasattr(value, 'isoformat'):
                    value = value.isoformat()
                elif hasattr(value, 'username'):
                    value = value.username
                data[field] = value
        return data
    
    @property
    def data(self):
        if hasattr(self.instance, '__iter__'):
            return [self.serialize_one(obj) for obj in self.instance]
        return self.serialize_one(self.instance)

# Usage
def api_articles(request):
    fields = request.GET.get('fields', '').split(',') or None
    articles = Article.objects.select_related('author').all()[:20]
    return JsonResponse({'data': ArticleSerializer(articles, fields=fields).data})
```

### Optimizing Queries

```python
def api_articles(request):
    requested = request.GET.get('fields', '').split(',')
    
    # Only select requested database columns
    db_fields = ['id']  # Always include id
    related = set()
    
    field_mapping = {
        'title': 'title',
        'body': 'body',
        'created_at': 'created_at',
        'views': 'views',
        'author': 'author__username',
    }
    
    for field in requested:
        if field in field_mapping:
            mapped = field_mapping[field]
            if '__' in mapped:
                related.add(mapped.split('__')[0])
            db_fields.append(mapped)
    
    articles = Article.objects.all()
    if related:
        articles = articles.select_related(*related)
    
    return JsonResponse({
        'data': list(articles.values(*db_fields)[:20])
    })
```

## Common Pitfalls

1. **Not validating fields**: Allowing `?fields=password` would be a security issue.

2. **Ignoring related objects**: Requesting `author` without `select_related()` causes N+1 queries.

3. **Breaking clients**: Once you publish available fields, removing them breaks existing clients.

## Best Practices

1. **Whitelist available fields**: Never expose internal or sensitive fields.
2. **Optimize database queries**: Use `only()`, `values()`, or `select_related()` based on requested fields.
3. **Document available fields**: Let clients know which fields can be requested.
4. **Provide sensible defaults**: Return commonly-needed fields when none specified.

## Summary

Sparse fieldsets reduce over-fetching by letting clients specify needed fields. Implement via query parameters, validate against a whitelist, and optimize database queries based on requested fields. This improves performance and gives clients flexibility.

## Resources

- [QuerySet only()](https://docs.djangoproject.com/en/6.0/ref/models/querysets/#only) — Loading only specific fields

---

> 📘 *This lesson is part of the [Django REST API Development](https://stanza.dev/courses/django-rest-api) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*