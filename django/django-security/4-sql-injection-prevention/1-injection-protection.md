---
source_course: "django-security"
source_lesson: "django-security-sql-injection-protection"
---

# SQL Injection Protection

## Introduction

SQL injection is one of the most dangerous and common web vulnerabilities. Attackers manipulate database queries by inserting malicious SQL code through user input. Django's ORM provides strong protection by automatically parameterizing all queries, but raw SQL usage requires careful handling.

## Key Concepts

- **SQL Injection**: Attack that inserts malicious SQL through user input to read, modify, or delete database data.
- **Parameterized Queries**: Technique where SQL structure and user values are sent separately, preventing injection.
- **Django ORM**: Object-Relational Mapper that automatically generates parameterized queries.
- **Raw SQL**: Direct SQL that bypasses ORM protection and requires manual parameterization.

## Real World Context

SQL injection has caused some of the largest data breaches in history, including the theft of millions of user records from major companies. Automated tools like sqlmap can exploit injection vulnerabilities in seconds. Even a single unparameterized query can expose your entire database, because attackers can use UNION SELECT to read any table.

## Deep Dive

### How SQL Injection Works

```python
# VULNERABLE CODE - Never do this!
username = request.GET.get('username')
query = f"SELECT * FROM users WHERE username = '{username}'"

# Attacker submits: ' OR '1'='1
# Resulting query:
SELECT * FROM users WHERE username = '' OR '1'='1'
# Returns ALL users!
```

### Django ORM Protection

Django's ORM automatically parameterizes queries:

```python
# SAFE - ORM parameterizes the query
user = User.objects.get(username=username)

# Under the hood, Django sends:
# SELECT * FROM users WHERE username = %s
# With parameters: ['user_input']

# The input is never interpolated into the query
```

### Safe Raw Queries

```python
# If you must use raw SQL, use parameters

# VULNERABLE - String formatting
User.objects.raw(f"SELECT * FROM users WHERE name = '{name}'")

# SAFE - Parameterized query
User.objects.raw("SELECT * FROM users WHERE name = %s", [name])

# SAFE - Named parameters
User.objects.raw(
    "SELECT * FROM users WHERE name = %(name)s",
    {'name': name}
)
```

### Direct Database Queries

```python
from django.db import connection

# VULNERABLE
with connection.cursor() as cursor:
    cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")

# SAFE - Use parameters
with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM users WHERE id = %s", [user_id])
    
    # For multiple parameters
    cursor.execute(
        "SELECT * FROM users WHERE status = %s AND role = %s",
        [status, role]
    )
```

### Extra() and RawSQL (Careful!)

```python
from django.db.models import F
from django.db.models.expressions import RawSQL

# VULNERABLE - Unparameterized
Article.objects.extra(
    where=[f"title LIKE '%{search}%'"]
)

# SAFE - Use ORM methods instead
Article.objects.filter(title__icontains=search)

# If RawSQL is necessary, parameterize it
Article.objects.annotate(
    custom_field=RawSQL(
        "(SELECT COUNT(*) FROM comments WHERE article_id = %s)",
        [article_id]
    )
)
```

### LIKE Patterns

```python
# Django handles LIKE escaping automatically
Article.objects.filter(title__contains=user_input)
Article.objects.filter(title__startswith=user_input)
Article.objects.filter(title__endswith=user_input)

# Special characters like % and _ are escaped
# User input: "100%" is safely searched as "100\%"
```

### Validating User Input

```python
# Even with ORM protection, validate input types

def get_user(request):
    try:
        user_id = int(request.GET.get('id'))
    except (TypeError, ValueError):
        return HttpResponseBadRequest('Invalid ID')
    
    user = get_object_or_404(User, pk=user_id)
    return render(request, 'user.html', {'user': user})

# Use forms for complex validation
class SearchForm(forms.Form):
    query = forms.CharField(max_length=100)
    category = forms.ChoiceField(choices=CATEGORY_CHOICES)
```

### Aggregation Safety

```python
from django.db.models import Count, Sum, F, Value
from django.db.models.functions import Coalesce

# SAFE - ORM aggregations are parameterized
Article.objects.annotate(
    comment_count=Count('comments')
).filter(comment_count__gt=10)

# SAFE - F() expressions
Article.objects.filter(views__gt=F('min_views'))

# SAFE - Value for literals
Article.objects.annotate(
    status_label=Coalesce('status', Value('unknown'))
)
```

## Common Pitfalls

1. **Using f-strings or .format() in raw SQL** — Even experienced developers fall back to string interpolation out of habit; always use %s placeholders.
2. **Assuming ORM is always enough** — Complex queries sometimes require raw SQL, but that raw SQL still needs parameterization.
3. **Forgetting to escape LIKE wildcards** — The % and _ characters have special meaning in LIKE queries and must be escaped in raw SQL.

## Best Practices

1. **Use the ORM for everything possible** — It parameterizes automatically and handles escaping edge cases.
2. **Always parameterize raw SQL** — Use %s placeholders with a values list, never string formatting.
3. **Validate input types early** — Coerce IDs to integers, validate choices against allowlists before they reach the database layer.

## Summary

- Django's ORM automatically prevents SQL injection through parameterized queries.
- Raw SQL must always use %s placeholders with separate value parameters.
- Validate and type-check user input even when using the ORM for defense in depth.

## Code Examples

**Comparison of dangerous string interpolation vs safe parameterized queries using Django ORM and raw SQL**

```python
from django.db import connection

# DANGEROUS: String interpolation in SQL
# cursor.execute(f"SELECT * FROM users WHERE name = '{name}'")

# SAFE: ORM (automatic parameterization)
user = User.objects.get(username=name)

# SAFE: Raw SQL with %s placeholders
with connection.cursor() as cursor:
    cursor.execute(
        "SELECT * FROM users WHERE name = %s AND active = %s",
        [name, True]
    )
```


## Resources

- [SQL Injection Protection](https://docs.djangoproject.com/en/6.0/topics/security/#sql-injection-protection) — Django SQL injection protection

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*