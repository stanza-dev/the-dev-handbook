---
source_course: "django-security"
source_lesson: "django-security-extra-expressions"
---

# Safe Use of Extra and Expressions

## Introduction

Django's extra(), RawSQL, and annotations can introduce SQL injection if misused. Learn the safe patterns.

## Key Concepts

**extra()**: Deprecated method for adding SQL fragments to queries.

**RawSQL()**: Expression for raw SQL in annotations/filters.

**F() and expressions**: Safe ORM alternatives to raw SQL.

## Deep Dive

### Avoid extra() - Use Expressions

```python
# DANGEROUS - extra() is deprecated and risky
Article.objects.extra(where=[f"title LIKE '%{search}%'"])

# SAFE - use ORM filters
Article.objects.filter(title__icontains=search)

# SAFE - use expressions
from django.db.models import F, Value
from django.db.models.functions import Concat

Article.objects.annotate(
    full_title=Concat('category__name', Value(': '), 'title')
)
```

### Safe RawSQL Usage

```python
from django.db.models.expressions import RawSQL

# DANGEROUS
Article.objects.annotate(
    rank=RawSQL(f"ts_rank(search_vector, '{query}')")
)

# SAFE - with parameters
Article.objects.annotate(
    rank=RawSQL("ts_rank(search_vector, %s)", [query])
)
```

### Prefer Built-in Expressions

```python
from django.db.models import F, Case, When, Value
from django.db.models.functions import Coalesce, Lower

# Instead of raw SQL for calculations
Article.objects.annotate(
    popularity=F('views') * 2 + F('comments_count')
)

# Conditional expressions
Article.objects.annotate(
    status_label=Case(
        When(is_published=True, then=Value('Live')),
        default=Value('Draft')
    )
)
```

## Best Practices

1. **Avoid extra()**: It's deprecated—use expressions instead.
2. **RawSQL with params**: Always pass values as parameters.
3. **Use database functions**: Django has many built-in functions.

## Summary

Prefer Django's expression API (F, Case, When, database functions) over raw SQL. When RawSQL is necessary, always use parameterized queries. Avoid the deprecated extra() method entirely.

## Resources

- [Query Expressions](https://docs.djangoproject.com/en/6.0/ref/models/expressions/) — Django query expressions

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*