---
source_course: "django-security"
source_lesson: "django-security-sql-injection-protection"
---

# SQL Injection Protection

SQL injection attacks manipulate database queries by inserting malicious SQL code.

## How SQL Injection Works

```python
# VULNERABLE CODE - Never do this!
username = request.GET.get('username')
query = f"SELECT * FROM users WHERE username = '{username}'"

# Attacker submits: ' OR '1'='1
# Resulting query:
SELECT * FROM users WHERE username = '' OR '1'='1'
# Returns ALL users!
```

## Django ORM Protection

Django's ORM automatically parameterizes queries:

```python
# SAFE - ORM parameterizes the query
user = User.objects.get(username=username)

# Under the hood, Django sends:
# SELECT * FROM users WHERE username = %s
# With parameters: ['user_input']

# The input is never interpolated into the query
```

## Safe Raw Queries

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

## Direct Database Queries

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

## Extra() and RawSQL (Careful!)

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

## LIKE Patterns

```python
# Django handles LIKE escaping automatically
Article.objects.filter(title__contains=user_input)
Article.objects.filter(title__startswith=user_input)
Article.objects.filter(title__endswith=user_input)

# Special characters like % and _ are escaped
# User input: "100%" is safely searched as "100\%"
```

## Validating User Input

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

## Aggregation Safety

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

## Resources

- [SQL Injection Protection](https://docs.djangoproject.com/en/6.0/topics/security/#sql-injection-protection) — Django SQL injection protection

---

> 📘 *This lesson is part of the [Django Security Best Practices](https://stanza.dev/courses/django-security) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*