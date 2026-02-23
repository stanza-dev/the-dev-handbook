---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-select-related"
---

# select_related for Foreign Keys

`select_related` performs a SQL JOIN to fetch related objects in a single query, eliminating the N+1 query problem for ForeignKey and OneToOneField relationships.

## The N+1 Problem

```python
# BAD: N+1 queries (1 query + N queries for authors)
articles = Article.objects.all()  # 1 query
for article in articles:
    print(article.author.name)  # N queries!
```

With 100 articles, this makes **101 queries**.

## The Solution: select_related

```python
# GOOD: Single query with JOIN
articles = Article.objects.select_related('author')  # 1 query
for article in articles:
    print(article.author.name)  # No additional queries
```

Generated SQL:
```sql
SELECT article.*, author.*
FROM article
INNER JOIN author ON article.author_id = author.id
```

## When to Use select_related

✅ **Use for:**
- ForeignKey fields
- OneToOneField fields
- When you'll access the related object

❌ **Don't use for:**
- ManyToManyField (use prefetch_related)
- Reverse ForeignKey relations (use prefetch_related)

## Multiple Relations

```python
# Follow multiple foreign keys
articles = Article.objects.select_related(
    'author',
    'category',
    'editor'
)
```

## Chained Relations

Follow relationships through multiple levels:

```python
# Article → Author → Profile
articles = Article.objects.select_related('author__profile')

for article in articles:
    print(article.author.profile.bio)  # No additional queries

# Multiple levels
Article.objects.select_related(
    'author__profile',
    'author__company',
    'category__parent'
)
```

## Combining with Other Methods

```python
# With filter
Article.objects.select_related('author').filter(is_published=True)

# With order_by
Article.objects.select_related('author').order_by('-pub_date')

# With values (the JOIN still happens, but prefetched data is redundant/unused)
Article.objects.select_related('author').values('title', 'author__name')
```

## Clearing select_related

```python
# If a manager has select_related by default
articles = Article.objects.select_related('author')

# Clear it
articles = articles.select_related(None)
```

## Performance Comparison

```python
# Measure with Django Debug Toolbar or:
import time
from django.db import connection

# Without select_related
start = time.time()
articles = Article.objects.all()[:100]
for a in articles:
    _ = a.author.name
print(f"Without: {len(connection.queries)} queries, {time.time()-start:.3f}s")

# With select_related
connection.queries.clear()
start = time.time()
articles = Article.objects.select_related('author')[:100]
for a in articles:
    _ = a.author.name
print(f"With: {len(connection.queries)} queries, {time.time()-start:.3f}s")
```

## Real-World Example

```python
# E-commerce order detail
order = Order.objects.select_related(
    'customer',
    'customer__profile',
    'shipping_address',
    'billing_address'
).get(pk=order_id)

# Access all without additional queries
print(order.customer.email)
print(order.customer.profile.phone)
print(order.shipping_address.street)
print(order.billing_address.city)
```\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test select_related for foreign keys with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some select_related for foreign keys features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- select_related for Foreign Keys is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Key example from select_related for Foreign Keys**

```python
# BAD: N+1 queries (1 query + N queries for authors)
articles = Article.objects.all()  # 1 query
for article in articles:
    print(article.author.name)  # N queries!
```


## Resources

- [select_related](https://docs.djangoproject.com/en/6.0/ref/models/querysets/#select-related) — Official select_related documentation

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*