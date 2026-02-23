---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-advanced-lookups"
---

# Advanced Field Lookups

Django provides many field lookups beyond the basics. Master these to write precise, efficient queries.

## String Lookups

```python
# Exact match (default)
Article.objects.filter(title='Django Guide')
Article.objects.filter(title__exact='Django Guide')  # Same

# Case-insensitive exact
Article.objects.filter(title__iexact='django guide')

# Contains (case-sensitive)
Article.objects.filter(title__contains='Django')

# Contains (case-insensitive)
Article.objects.filter(title__icontains='django')

# Starts with
Article.objects.filter(title__startswith='How')
Article.objects.filter(title__istartswith='how')  # Case-insensitive

# Ends with
Article.objects.filter(title__endswith='Guide')
Article.objects.filter(title__iendswith='guide')  # Case-insensitive

# Regular expression (database-dependent)
Article.objects.filter(title__regex=r'^(How|What|Why)')
Article.objects.filter(title__iregex=r'^(how|what|why)')  # Case-insensitive
```

## Comparison Lookups

```python
# Greater than / Less than
Product.objects.filter(price__gt=100)      # > 100
Product.objects.filter(price__gte=100)     # >= 100
Product.objects.filter(price__lt=50)       # < 50
Product.objects.filter(price__lte=50)      # <= 50

# Range (inclusive)
Product.objects.filter(price__range=(10, 100))  # 10 <= price <= 100

# In a list
Article.objects.filter(status__in=['published', 'featured'])
Article.objects.filter(pk__in=[1, 2, 3, 4, 5])

# In a subquery
active_author_ids = Author.objects.filter(is_active=True).values('id')
Article.objects.filter(author_id__in=active_author_ids)
```

## Date and Time Lookups

```python
from datetime import date, datetime

# Extract parts
Article.objects.filter(pub_date__year=2024)
Article.objects.filter(pub_date__month=6)
Article.objects.filter(pub_date__day=15)
Article.objects.filter(pub_date__week=25)  # ISO week number
Article.objects.filter(pub_date__week_day=2)  # Sunday=1, Saturday=7
Article.objects.filter(pub_date__quarter=2)  # Q2

# Time parts (for DateTimeField)
Event.objects.filter(start_time__hour=14)
Event.objects.filter(start_time__minute=30)
Event.objects.filter(start_time__second=0)

# Date comparison
Article.objects.filter(pub_date__date=date.today())
Article.objects.filter(pub_date__date__gte=date(2024, 1, 1))

# Time comparison
Event.objects.filter(start_time__time__gte=datetime.strptime('09:00', '%H:%M').time())
```

## Null and Boolean Lookups

```python
# Is null
Article.objects.filter(published_at__isnull=True)   # Not published
Article.objects.filter(published_at__isnull=False)  # Published

# Boolean
Article.objects.filter(is_featured=True)
Article.objects.filter(is_featured__exact=False)
```

## Relationship Lookups

```python
# Span relationships with double underscore
Article.objects.filter(author__name='John')
Article.objects.filter(author__profile__country='USA')
Article.objects.filter(tags__name='python')

# Check related objects exist
Author.objects.filter(article__isnull=False).distinct()  # Has articles
Author.objects.filter(article__isnull=True)  # No articles
```

## JSON Field Lookups (PostgreSQL)

```python
# Model with JSONField
class Product(models.Model):
    data = models.JSONField()

# Key lookup
Product.objects.filter(data__color='red')
Product.objects.filter(data__dimensions__width__gt=10)

# Contains key
Product.objects.filter(data__has_key='color')
Product.objects.filter(data__has_keys=['color', 'size'])
Product.objects.filter(data__has_any_keys=['color', 'material'])

# Contains value
Product.objects.filter(data__contains={'color': 'red'})
Product.objects.filter(data__contained_by={'color': 'red', 'size': 'large'})
```

## Custom Lookups

```python
from django.db.models import Lookup
from django.db.models.fields import Field


class NotEqual(Lookup):
    lookup_name = 'ne'

    def as_sql(self, compiler, connection):
        lhs, lhs_params = self.process_lhs(compiler, connection)
        rhs, rhs_params = self.process_rhs(compiler, connection)
        return f'{lhs} <> {rhs}', lhs_params + rhs_params


Field.register_lookup(NotEqual)

# Usage
Product.objects.filter(status__ne='deleted')
```\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test advanced field lookups with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some advanced field lookups features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- Advanced Field Lookups is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Advanced field lookups: range, regex, and subquery-based in**

```python
# Range lookup
from datetime import date
Event.objects.filter(
    start_date__range=(date(2025, 1, 1), date(2025, 12, 31))
)

# Regex lookup
Product.objects.filter(sku__regex=r'^[A-Z]{2}-\d{4}$')

# In subquery
active_author_ids = Author.objects.filter(is_active=True).values('id')
Article.objects.filter(author_id__in=active_author_ids)
```


## Resources

- [QuerySet Field Lookups](https://docs.djangoproject.com/en/6.0/ref/models/querysets/#field-lookups) — Complete reference of all field lookups

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*