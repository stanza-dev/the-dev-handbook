---
source_course: "django-performance"
source_lesson: "django-performance-query-optimization"
---

# QuerySet Optimization

## Introduction

Database queries are often the biggest performance bottleneck in Django applications. A single page load can trigger hundreds of unnecessary queries if your code is not optimized. Learning to reduce query count and use the ORM efficiently is the highest-leverage performance skill you can develop.

## Key Concepts

- **N+1 Problem**: When accessing related objects in a loop triggers one additional query per object, leading to N+1 total queries.
- **select_related**: Performs a SQL JOIN to fetch related ForeignKey/OneToOne objects in a single query.
- **prefetch_related**: Executes a separate query for ManyToMany/reverse FK relationships and joins in Python.
- **Bulk Operations**: Methods like `bulk_create()` and `bulk_update()` that batch multiple database operations into fewer queries.

## Real World Context

In production, the N+1 problem is the number one cause of slow page loads. A blog listing page showing 50 articles with their authors generates 51 queries without `select_related` but only 1 with it. E-commerce sites use `prefetch_related` with `Prefetch` objects to load product variants, images, and reviews efficiently. Companies like Instagram and Disqus have shared how query optimization reduced their response times by 10-100x.

## Deep Dive

The Django ORM provides several powerful tools to optimize database access. The key is understanding which method to use for each relationship type and access pattern.


## Introduction

Database queries are often the biggest performance bottleneck. Understanding how to optimize them is crucial.

## The N+1 Problem

The most common performance issue is the N+1 query problem:

```python
# BAD: N+1 queries (1 for articles + N for each author)
articles = Article.objects.all()
for article in articles:
    print(article.author.username)  # Each access hits the database!
```

## select_related

Use `select_related` for ForeignKey and OneToOneField relationships:

```python
# GOOD: Single query with JOIN
articles = Article.objects.select_related('author', 'category').all()
for article in articles:
    print(article.author.username)  # No additional queries!
    print(article.category.name)
```

## prefetch_related

Use `prefetch_related` for ManyToManyField and reverse ForeignKey:

```python
# GOOD: 2 queries (articles + tags)
articles = Article.objects.prefetch_related('tags').all()
for article in articles:
    for tag in article.tags.all():  # No additional queries!
        print(tag.name)
```

## Prefetch Objects

```python
from django.db.models import Prefetch

# Custom prefetch with filtering
articles = Article.objects.prefetch_related(
    Prefetch(
        'comments',
        queryset=Comment.objects.filter(is_approved=True).order_by('-created_at')[:5],
        to_attr='recent_comments'  # Access via article.recent_comments
    )
)

for article in articles:
    for comment in article.recent_comments:
        print(comment.body)
```

## Only and Defer

```python
# Only load specific fields
articles = Article.objects.only('title', 'slug', 'pub_date')

# Defer loading of specific fields (opposite of only)
articles = Article.objects.defer('body', 'html_content')
```

## Values and Values List

```python
# Return dictionaries instead of model instances
articles = Article.objects.values('title', 'author__username')
# [{'title': 'Article 1', 'author__username': 'john'}, ...]

# Return tuples
titles = Article.objects.values_list('title', flat=True)
# ['Article 1', 'Article 2', ...]
```

## Aggregation Optimization

```python
from django.db.models import Count, Avg, Sum, F

# Single query for aggregation
stats = Article.objects.aggregate(
    total=Count('id'),
    avg_views=Avg('views'),
    total_views=Sum('views')
)

# Annotation for per-object calculations
authors = Author.objects.annotate(
    article_count=Count('articles'),
    total_views=Sum('articles__views')
).filter(article_count__gt=5)
```

## Exists vs Count

```python
# BAD: Counts all rows even though we just need to know if any exist
if Article.objects.filter(status='published').count() > 0:
    pass

# GOOD: Stops at first match
if Article.objects.filter(status='published').exists():
    pass
```

## Bulk Operations

```python
# BAD: N insert queries
for item in items:
    Article.objects.create(**item)

# GOOD: Single bulk insert
Article.objects.bulk_create([
    Article(**item) for item in items
], batch_size=1000)

# Bulk update
Article.objects.filter(status='draft').update(status='published')

# Bulk update with different values
articles_to_update = []
for article in articles:
    article.views += 1
    articles_to_update.append(article)
Article.objects.bulk_update(articles_to_update, ['views'], batch_size=1000)
```

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Using `select_related` for ManyToMany fields** — This will raise an error. Use `prefetch_related` for ManyToMany and reverse ForeignKey relationships instead.
2. **Calling `.count()` when `.exists()` suffices** — `count()` scans all matching rows while `exists()` stops at the first match. Always use `exists()` for boolean checks.
3. **Forgetting `batch_size` in `bulk_create`** — Without it, Django tries to insert all objects in a single query which can exceed database limits for very large datasets.

## Best Practices

1. **Use Django Debug Toolbar in development** — It shows the exact number and content of queries per request, making N+1 problems immediately visible.
2. **Add `select_related`/`prefetch_related` in managers** — Define a custom manager with default optimizations so every query benefits.
3. **Use `values()` or `values_list()` when you only need specific fields** — This avoids instantiating full model objects and reduces memory usage.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Using select_related to solve the N+1 query problem with a single SQL JOIN**

```python
# BAD: N insert queries
for item in items:
    Article.objects.create(**item)

# GOOD: Single bulk insert
Article.objects.bulk_create([
    Article(**item) for item in items
], batch_size=1000)

# Bulk update
Article.objects.filter(status='draft').update(status='published')

# Bulk update with different values
articles_to_update = []
for article in articles:
    article.views += 1
    articles_to_update.append(article)
Article.objects.bulk_update(articles_to_update, ['views'], batch_size=1000)
```


## Resources

- [Database Optimization](https://docs.djangoproject.com/en/6.0/topics/db/optimization/) — Official database optimization guide

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*