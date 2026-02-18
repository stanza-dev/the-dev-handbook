---
source_course: "django-performance"
source_lesson: "django-performance-query-debugging"
---

# Query Debugging Techniques

## Introduction

Identifying slow queries is the first step to optimization. Django provides tools to inspect and analyze database queries.

## Key Concepts

**Query Logging**: Capture all SQL queries executed.

**EXPLAIN**: Database query execution plan analysis.

## Deep Dive

### Viewing Generated SQL

```python
# Print the SQL query
qs = Article.objects.filter(status='published')
print(qs.query)

# With Django Debug Toolbar (development)
# Shows all queries in browser panel
```

### Using EXPLAIN

```python
# PostgreSQL query plan
qs = Article.objects.filter(status='published')
print(qs.explain())

# With analyze for actual timing
print(qs.explain(analyze=True))

# Output shows:
# - Seq Scan vs Index Scan
# - Estimated rows
# - Actual execution time
```

### Query Counting

```python
from django.db import connection, reset_queries
from django.conf import settings

# Enable query logging
settings.DEBUG = True
reset_queries()

# Your code here
articles = Article.objects.all()
for a in articles:
    print(a.author.name)  # N+1!

print(f'Queries: {len(connection.queries)}')
for q in connection.queries:
    print(f"{q['time']}s: {q['sql'][:80]}")
```

### Django Debug Toolbar

```python
# settings.py
INSTALLED_APPS = ['debug_toolbar', ...]
MIDDLEWARE = ['debug_toolbar.middleware.DebugToolbarMiddleware', ...]
INTERNAL_IPS = ['127.0.0.1']
```

## Best Practices

1. **Use DEBUG=True only in development**: Query logging adds overhead.
2. **Check EXPLAIN for slow queries**: Look for Seq Scans on large tables.
3. **Count queries in tests**: Catch N+1 problems early.

## Summary

Use query logging and EXPLAIN to identify slow queries. Django Debug Toolbar is invaluable for development. Count queries in tests to prevent regressions.

## Resources

- [QuerySet.explain()](https://docs.djangoproject.com/en/6.0/ref/models/querysets/#explain) — Django explain documentation

---

> 📘 *This lesson is part of the [Django Performance & Optimization](https://stanza.dev/courses/django-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*