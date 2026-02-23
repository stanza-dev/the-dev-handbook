---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-prefetch-related"
---

# prefetch_related for Many Relations

`prefetch_related` fetches related objects in separate queries and does the joining in Python. It's designed for ManyToMany and reverse ForeignKey relations.

## When to Use prefetch_related

✅ **Use for:**
- ManyToManyField
- Reverse ForeignKey (e.g., `author.article_set`)
- GenericRelation

## Basic Usage

```python
# ManyToMany: Articles have many tags
articles = Article.objects.prefetch_related('tags')
for article in articles:
    for tag in article.tags.all():  # No additional queries
        print(tag.name)

# Reverse ForeignKey: Author has many articles
authors = Author.objects.prefetch_related('article_set')
for author in authors:
    for article in author.article_set.all():  # No additional queries
        print(article.title)
```

## How It Works

```python
# prefetch_related makes 2 queries:
authors = Author.objects.prefetch_related('article_set')

# Query 1: SELECT * FROM author
# Query 2: SELECT * FROM article WHERE author_id IN (1, 2, 3, ...)
# Then joins in Python
```

## Combining with select_related

```python
# Optimize all relationships
articles = Article.objects.select_related(
    'author',         # ForeignKey - JOIN
    'category'        # ForeignKey - JOIN
).prefetch_related(
    'tags',           # ManyToMany - separate query
    'comments'        # Reverse FK - separate query
)
```

## Prefetch Object for Custom Queries

Use `Prefetch` for more control:

```python
from django.db.models import Prefetch

# Filter prefetched objects
authors = Author.objects.prefetch_related(
    Prefetch(
        'article_set',
        queryset=Article.objects.filter(is_published=True)
    )
)

# Change the attribute name
authors = Author.objects.prefetch_related(
    Prefetch(
        'article_set',
        queryset=Article.objects.filter(is_published=True),
        to_attr='published_articles'  # New attribute name
    )
)

for author in authors:
    for article in author.published_articles:  # List, not queryset!
        print(article.title)
```

## Complex Prefetch Examples

### Nested Prefetch

The following example demonstrates how to use nested prefetch in practice:

```python
# Authors → Articles → Comments
authors = Author.objects.prefetch_related(
    Prefetch(
        'article_set',
        queryset=Article.objects.prefetch_related('comments')
    )
)
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Prefetch with select_related

The following example demonstrates how to use prefetch with select_related in practice:

```python
# Articles with their author (select) and author's other articles (prefetch)
articles = Article.objects.select_related('author').prefetch_related(
    Prefetch(
        'author__article_set',
        queryset=Article.objects.filter(is_published=True).order_by('-pub_date')[:5],
        to_attr='recent_articles'
    )
)
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Ordered Prefetch

The following example demonstrates how to use ordered prefetch in practice:

```python
# Get latest comments only
articles = Article.objects.prefetch_related(
    Prefetch(
        'comment_set',
        queryset=Comment.objects.order_by('-created_at')[:5],
        to_attr='recent_comments'
    )
)
```

## Important: Don't Break the Cache

```python
# GOOD: Uses prefetched data
articles = Article.objects.prefetch_related('tags')
for article in articles:
    tags = article.tags.all()  # Uses cache

# BAD: Filter breaks the cache, new query!
for article in articles:
    tags = article.tags.filter(active=True)  # New query!

# SOLUTION: Use Prefetch with filtered queryset
articles = Article.objects.prefetch_related(
    Prefetch(
        'tags',
        queryset=Tag.objects.filter(active=True),
        to_attr='active_tags'
    )
)
for article in articles:
    tags = article.active_tags  # Uses cache
```

## Performance Tips

1. **Use Prefetch for filtering**: Avoid filtering on prefetched relations
2. **to_attr returns a list**: Not a queryset, so it's already evaluated
3. **Limit prefetched items**: Use `[:n]` in Prefetch queryset
4. **Combine wisely**: select_related for FK, prefetch_related for M2M/reverse\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test prefetch_related for many relations with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some prefetch_related for many relations features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- prefetch_related for Many Relations is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Key example from prefetch_related for Many Relations**

```python
# ManyToMany: Articles have many tags
articles = Article.objects.prefetch_related('tags')
for article in articles:
    for tag in article.tags.all():  # No additional queries
        print(tag.name)

# Reverse ForeignKey: Author has many articles
authors = Author.objects.prefetch_related('article_set')
for author in authors:
    for article in author.article_set.all():  # No additional queries
        print(article.title)
```


## Resources

- [prefetch_related](https://docs.djangoproject.com/en/6.0/ref/models/querysets/#prefetch-related) — Official prefetch_related documentation

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*