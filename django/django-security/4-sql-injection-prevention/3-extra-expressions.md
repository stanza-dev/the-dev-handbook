---
source_course: "django-security"
source_lesson: "django-security-extra-expressions"
---

# Safe Use of Extra and Expressions

## Introduction

Django's extra(), RawSQL, and annotations can introduce SQL injection if misused. Learn the safe patterns.

## Key Concepts

**extra()**: Legacy method for adding SQL fragments to queries (discouraged in favor of expressions).

**RawSQL()**: Expression for raw SQL in annotations/filters.

**F() and expressions**: Safe ORM alternatives to raw SQL.



## Real World Context

Django's expression API has grown significantly since extra() was introduced. Teams maintaining legacy codebases often have extra() calls that were safe when written but become injection vectors when someone modifies them to accept user input. Migrating to F(), Case(), and database functions eliminates this risk.

## Deep Dive

### Avoid extra() - Use Expressions

```python
# DANGEROUS - extra() is a legacy API and risky
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



## Common Pitfalls

1. **Modifying old extra() calls to accept user input** — Legacy extra() calls that used hardcoded values become vulnerable when refactored to use variables; replace with ORM expressions instead.
2. **Forgetting to parameterize RawSQL annotations** — RawSQL() without parameters is just as vulnerable as an f-string query; always pass values as the second argument.

## Best Practices

1. **Avoid extra()**: It's a legacy API—use expressions instead.
2. **RawSQL with params**: Always pass values as parameters.
3. **Use database functions**: Django has many built-in functions.

## Summary

Prefer Django's expression API (F, Case, When, database functions) over raw SQL. When RawSQL is necessary, always use parameterized queries. Avoid the legacy extra() method entirely.

## Resources

- [Query Expressions](https://docs.djangoproject.com/en/6.0/ref/models/expressions/) — Django query expressions

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*