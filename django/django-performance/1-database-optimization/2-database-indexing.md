---
source_course: "django-performance"
source_lesson: "django-performance-database-indexing"
---

# Database Indexing

Indexes dramatically speed up queries but have trade-offs.

## Basic Index

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    slug = models.SlugField(db_index=True)  # Simple index
    status = models.CharField(max_length=20, db_index=True)
    pub_date = models.DateTimeField(db_index=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE)  # Auto-indexed
```

## Meta Indexes

```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    pub_date = models.DateTimeField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    
    class Meta:
        indexes = [
            # Single column index
            models.Index(fields=['slug']),
            
            # Composite index (order matters!)
            models.Index(fields=['status', 'pub_date']),
            
            # Descending index
            models.Index(fields=['-pub_date']),
            
            # Named index
            models.Index(
                fields=['status', '-pub_date'],
                name='article_status_date_idx'
            ),
            
            # Partial index (PostgreSQL)
            models.Index(
                fields=['pub_date'],
                condition=Q(status='published'),
                name='published_articles_idx'
            ),
        ]
```

## Covering Indexes (PostgreSQL)

```python
class Article(models.Model):
    class Meta:
        indexes = [
            # Include additional columns for index-only scans
            models.Index(
                fields=['status'],
                include=['title', 'pub_date'],
                name='status_covering_idx'
            ),
        ]
```

## When to Index

```python
# Index columns used in:
# - WHERE clauses (filter, exclude)
# - ORDER BY clauses
# - JOIN conditions (ForeignKey)
# - GROUP BY clauses

# Common query patterns to optimize:
Article.objects.filter(status='published')  # Index status
Article.objects.filter(status='published').order_by('-pub_date')  # Composite index
Article.objects.filter(author=user)  # ForeignKey auto-indexed
```

## Analyzing Queries

```python
# Django Debug Toolbar shows queries automatically

# Manual query inspection
qs = Article.objects.filter(status='published').select_related('author')
print(qs.query)  # See generated SQL

# Explain query (PostgreSQL)
print(qs.explain())  # See query execution plan
print(qs.explain(analyze=True))  # With actual timing
```

## Index Trade-offs

```python
# Indexes speed up reads but slow down writes
# Each INSERT/UPDATE must also update indexes

# Good candidates for indexing:
# - Frequently queried columns
# - High cardinality columns (many unique values)
# - Columns in WHERE, ORDER BY, JOIN

# Poor candidates:
# - Rarely queried columns
# - Low cardinality columns (few unique values)
# - Frequently updated columns
# - Very wide columns (text, JSON)
```

## Resources

- [Model Indexes](https://docs.djangoproject.com/en/6.0/ref/models/indexes/) — Django model index reference

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*