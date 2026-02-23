---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-database-indexes"
---

# Database Indexes and Meta.indexes

## Introduction

Indexes are the single most impactful optimization for database read performance. Django's `Meta.indexes` lets you declaratively define indexes on your models, and migrations handle the rest.

## Key Concepts

- **Index**: A database structure that speeds up lookups on specific columns at the cost of additional storage and slower writes.
- **B-tree Index**: The default index type — efficient for equality, range, and ordering queries.
- **Composite Index**: An index on multiple columns — the column order matters for query matching.
- **Partial Index**: An index with a `condition` that only includes rows matching a filter.

## Real World Context

A table with 10 million rows where `status='active'` is filtered on every request can go from 500ms to 2ms with the right index. Most Django performance issues in production trace back to missing indexes.

## Deep Dive

### Declaring Indexes

Declaring indexes in Meta tells Django to create database indexes during migration, which dramatically speeds up filtered queries:

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    pub_date = models.DateTimeField()
    author = models.ForeignKey('Author', on_delete=models.CASCADE)

    class Meta:
        indexes = [
            # Single-column index
            models.Index(fields=['status'], name='article_status_idx'),
            # Composite index (column order matters!)
            models.Index(fields=['status', '-pub_date'], name='article_status_date_idx'),
            # Partial index (PostgreSQL, SQLite 3.9+)
            models.Index(
                fields=['pub_date'],
                name='article_published_idx',
                condition=models.Q(status='published')
            ),
        ]
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### How Composite Indexes Work

Declaring indexes in Meta tells Django to create database indexes during migration, which dramatically speeds up filtered queries:

```python
# This index: ['status', '-pub_date']

# USES the index (leftmost prefix):
Article.objects.filter(status='published')
Article.objects.filter(status='published').order_by('-pub_date')
Article.objects.filter(status='published', pub_date__gte=cutoff)

# DOES NOT use the index:
Article.objects.filter(pub_date__gte=cutoff)  # Skips 'status'
Article.objects.order_by('pub_date')  # Wrong column order
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Verifying Index Usage

Declaring indexes in Meta tells Django to create database indexes during migration, which dramatically speeds up filtered queries:

```python
# Use .explain() to check if your index is used
qs = Article.objects.filter(status='published').order_by('-pub_date')
print(qs.explain(verbose=True))
# Look for: Index Scan using article_status_date_idx
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Unique Indexes

Declaring indexes in Meta tells Django to create database indexes during migration, which dramatically speeds up filtered queries:

```python
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    username_slug = models.SlugField()

    class Meta:
        constraints = [
            models.UniqueConstraint(
                fields=['username_slug'],
                name='unique_username_slug'
            ),
        ]
```

## Common Pitfalls

1. **Indexing every column** — Each index slows down INSERT/UPDATE operations. Only index columns used in WHERE, ORDER BY, and JOIN clauses.
2. **Wrong composite index order** — `['status', 'date']` helps `filter(status=X)` but not `filter(date=Y)`. Put the most selective column first.
3. **Ignoring partial indexes** — A full index on a column where 90% of queries filter `status='active'` wastes space. Use a conditional index.

## Best Practices

1. **Profile before indexing** — Use `.explain()` and `django-debug-toolbar` to identify slow queries, then add targeted indexes.
2. **Use partial indexes for filtered queries** — If you always filter on `status='published'`, a partial index is smaller and faster.
3. **Review indexes periodically** — Remove unused indexes that slow down writes without benefiting reads.

## Summary

- Define indexes in `Meta.indexes` — Django handles migration generation.
- Composite index column order determines which queries benefit.
- Partial indexes (with `condition`) are smaller and faster for filtered queries.
- Use `.explain()` to verify index usage.
- Balance read speed against write overhead — don't over-index.

## Code Examples

**Composite and partial indexes declared in Meta.indexes**

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    pub_date = models.DateTimeField()

    class Meta:
        indexes = [
            models.Index(fields=['status', '-pub_date'], name='status_date_idx'),
            models.Index(
                fields=['pub_date'],
                name='published_date_idx',
                condition=models.Q(status='published')  # Partial index
            ),
        ]
```


## Resources

- [Model Index Reference](https://docs.djangoproject.com/en/6.0/ref/models/indexes/) — Official reference for Django model indexes

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*