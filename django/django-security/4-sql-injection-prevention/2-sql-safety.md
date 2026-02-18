---
source_course: "django-security"
source_lesson: "django-security-raw-sql-safety"
---

# Safe Raw SQL Queries

## Introduction

Sometimes the ORM isn't enough—complex queries, performance optimization, or database-specific features require raw SQL. Here's how to write raw queries safely.

## Key Concepts

**Parameterized Queries**: SQL with placeholders (%s) where values are passed separately.

**Raw Manager**: Model.objects.raw() for queries returning model instances.

**Database Cursor**: connection.cursor() for arbitrary queries.

## Deep Dive

### Using Model.objects.raw()

```python
# DANGEROUS - never do this
User.objects.raw(f"SELECT * FROM users WHERE name = '{name}'")

# SAFE - parameterized
User.objects.raw("SELECT * FROM users WHERE name = %s", [name])

# Named parameters
User.objects.raw(
    "SELECT * FROM users WHERE status = %(status)s AND role = %(role)s",
    {'status': 'active', 'role': 'admin'}
)
```

### Using Database Cursor

```python
from django.db import connection

# SAFE
with connection.cursor() as cursor:
    cursor.execute(
        "SELECT COUNT(*) FROM articles WHERE category_id = %s",
        [category_id]
    )
    count = cursor.fetchone()[0]

# Multiple parameters
cursor.execute(
    "UPDATE users SET status = %s WHERE last_login < %s",
    ['inactive', cutoff_date]
)
```

### Handling LIKE Patterns

```python
# User input with LIKE - escape special chars
search = search_term.replace('%', '\\%').replace('_', '\\_')
cursor.execute(
    "SELECT * FROM articles WHERE title LIKE %s",
    [f'%{search}%']
)
```

## Best Practices

1. **Always use placeholders**: Never f-strings or format().
2. **Prefer ORM**: Only use raw SQL when necessary.
3. **Escape LIKE wildcards**: % and _ have special meaning.

## Summary

Raw SQL is safe when using parameterized queries with %s placeholders. Never use string interpolation, always pass values as a separate list, and escape LIKE wildcards in search terms.

## Resources

- [Raw SQL Queries](https://docs.djangoproject.com/en/6.0/topics/db/sql/) — Django raw SQL documentation

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*