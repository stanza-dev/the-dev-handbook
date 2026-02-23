---
source_course: "django-performance"
source_lesson: "django-performance-database-indexing"
---

# Database Indexing

## Introduction

Database indexes are data structures that dramatically speed up query lookups at the cost of additional storage and slightly slower writes. Choosing the right indexes for your query patterns is one of the most impactful performance optimizations you can make.

## Key Concepts

- **Index**: A database data structure that speeds up lookups by maintaining a sorted reference to table rows.
- **Composite Index**: An index on multiple columns, where column order determines which queries benefit.
- **Partial Index**: An index that only covers rows matching a condition, reducing index size and maintenance cost.
- **Covering Index**: An index that includes all columns needed by a query, allowing the database to answer without reading the table.

## Real World Context

A common scenario: your article listing page filters by `status=published` and orders by `-pub_date`. Without a composite index on `(status, pub_date)`, PostgreSQL must scan the entire table and sort results. With the right index, the same query completes in milliseconds even with millions of rows. E-commerce sites with large product catalogs rely heavily on partial indexes to keep frequently-filtered published products fast.

## Deep Dive

Database indexes work like a book's index — they let the database jump directly to the relevant rows instead of scanning the entire table. Django provides several ways to define indexes through the ORM.


## Introduction

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

## Summary

- The techniques covered in this lesson are essential for production-quality applications.
- Always measure before and after optimizing to verify improvements.
- Start with the simplest approach and add complexity only when needed.

## Common Pitfalls

1. **Over-indexing** — Every index slows down INSERT, UPDATE, and DELETE operations because the index must also be updated. Only index columns that appear in frequent WHERE, ORDER BY, or JOIN clauses.
2. **Wrong column order in composite indexes** — An index on `(status, pub_date)` helps queries filtering by status, but NOT queries filtering only by pub_date. Put the most selective column first.
3. **Indexing low-cardinality columns** — A boolean `is_active` column with only True/False values rarely benefits from a regular index. Consider a partial index instead.

## Best Practices

1. **Use `EXPLAIN ANALYZE` to verify index usage** — After adding an index, confirm the query planner actually uses it. Sometimes PostgreSQL chooses a sequential scan if the table is small.
2. **Name your indexes explicitly** — Use the `name` parameter in `models.Index()` for easier identification in database tools and migration files.
3. **Review indexes periodically** — As query patterns change, old indexes may become unused while new patterns need coverage. Use `pg_stat_user_indexes` to find unused indexes.

## Summary

- The techniques covered in this lesson are essential for production-quality Django applications.
- Always measure and profile before optimizing to ensure you're addressing the actual bottleneck.
- Start with the simplest approach and add complexity only when monitoring shows it's needed.

## Code Examples

**Defining composite and partial indexes in Django model Meta class**

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
# ...
```


## Resources

- [Model Indexes](https://docs.djangoproject.com/en/6.0/ref/models/indexes/) — Django model index reference

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*