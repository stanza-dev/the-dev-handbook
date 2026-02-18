---
source_course: "django-performance"
source_lesson: "django-performance-query-optimization"
---

# QuerySet Optimization

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

## Resources

- [Database Optimization](https://docs.djangoproject.com/en/6.0/topics/db/optimization/) — Official database optimization guide

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*