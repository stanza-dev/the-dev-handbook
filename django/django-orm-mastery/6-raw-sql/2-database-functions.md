---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-database-functions"
---

# Database Functions

Django provides database functions that translate to SQL functions across different database backends. These are safer and more portable than raw SQL.

## String Functions

```python
from django.db.models.functions import (
    Concat, Lower, Upper, Length, Left, Right,
    LPad, RPad, Replace, Reverse, StrIndex, Substr, Trim
)
from django.db.models import Value

# Concatenate fields
Person.objects.annotate(
    full_name=Concat('first_name', Value(' '), 'last_name')
)

# Transform case
Person.objects.annotate(
    name_upper=Upper('first_name'),
    name_lower=Lower('last_name')
)

# Get length
Article.objects.annotate(
    title_length=Length('title')
).filter(title_length__gt=50)

# Substring
Person.objects.annotate(
    initials=Concat(
        Substr('first_name', 1, 1),
        Substr('last_name', 1, 1)
    )
)
```

## Date/Time Functions

```python
from django.db.models.functions import (
    Extract, ExtractYear, ExtractMonth, ExtractDay,
    ExtractHour, ExtractMinute, ExtractWeekDay,
    TruncDate, TruncMonth, TruncYear, TruncHour,
    Now
)

# Extract parts
Article.objects.annotate(
    year=ExtractYear('pub_date'),
    month=ExtractMonth('pub_date'),
    day=ExtractDay('pub_date')
)

# Generic extract
Article.objects.annotate(
    quarter=Extract('pub_date', 'quarter')
)

# Truncate to period
Article.objects.annotate(
    month=TruncMonth('pub_date')
).values('month').annotate(
    count=Count('id')
).order_by('month')

# Current timestamp
Article.objects.filter(pub_date__lte=Now())
```

## Math Functions

```python
from django.db.models.functions import (
    Abs, Ceil, Floor, Round, Sqrt, Power,
    Greatest, Least, Mod
)

# Absolute value
Account.objects.annotate(abs_balance=Abs('balance'))

# Rounding
Product.objects.annotate(
    rounded_price=Round('price', 2),
    ceiling_price=Ceil('price'),
    floor_price=Floor('price')
)

# Greatest/Least of multiple values
Product.objects.annotate(
    effective_price=Least('price', 'sale_price'),
    max_price=Greatest('price', 'original_price')
)
```

## Conditional Functions

```python
from django.db.models import Case, When, Value, F
from django.db.models.functions import Coalesce, NullIf

# Coalesce (first non-null value)
Person.objects.annotate(
    display_name=Coalesce('nickname', 'first_name', Value('Anonymous'))
)

# NullIf (return NULL if values are equal)
Product.objects.annotate(
    adjusted_price=NullIf('price', 0)  # NULL if price is 0
)

# Case/When (SQL CASE statement)
Person.objects.annotate(
    age_group=Case(
        When(age__lt=18, then=Value('Minor')),
        When(age__lt=65, then=Value('Adult')),
        default=Value('Senior')
    )
)
```

## Window Functions

```python
from django.db.models import F, Window
from django.db.models.functions import (
    Rank, DenseRank, RowNumber, 
    CumeDist, PercentRank, NthValue,
    Lag, Lead, FirstValue, LastValue
)

# Rank within partition
Product.objects.annotate(
    category_rank=Window(
        expression=Rank(),
        partition_by=[F('category')],
        order_by=F('price').desc()
    )
)

# Row number
Article.objects.annotate(
    row_num=Window(
        expression=RowNumber(),
        order_by=F('pub_date').desc()
    )
)

# Running total
Order.objects.annotate(
    running_total=Window(
        expression=Sum('amount'),
        order_by=F('created_at').asc()
    )
)

# Compare with previous row (two .annotate() calls needed)
# First: compute window function
queryset = StockPrice.objects.annotate(
    previous_price=Window(
        expression=Lag('price', 1),
        order_by=F('date').asc()
    )
)
# Second: reference the window result
queryset = queryset.annotate(
    price_change=F('price') - F('previous_price')
)
```

## Custom Database Functions

```python
from django.db.models import Func

# Create a custom function
class Sin(Func):
    function = 'SIN'

class Log(Func):
    function = 'LOG'
    arity = 2  # Two arguments: LOG(base, value)

# Use it
Point.objects.annotate(
    y=Sin(F('angle'))
)
```\n\n## Common Pitfalls\n\n1. **Not testing edge cases** — Always test database functions with empty querysets, NULL values, and boundary conditions.\n2. **Premature optimization** — Profile queries with `.explain()` before applying complex optimizations.\n3. **Ignoring database-specific behavior** — Some database functions features behave differently across PostgreSQL, MySQL, and SQLite.\n\n## Best Practices\n\n1. **Keep queries readable** — Use meaningful variable names and chain methods logically.\n2. **Test with realistic data** — Create fixtures that match production data patterns for accurate performance testing.\n3. **Document complex queries** — Add comments explaining the business logic behind non-obvious query patterns.\n\n## Summary\n\n- Database Functions is a core Django ORM feature for building efficient database queries.\n- Always consider query performance and use `.explain()` to verify query plans.\n- Test edge cases including empty results, NULL values, and large datasets.\n- Refer to the Django documentation for database-specific behavior and limitations.

## Code Examples

**Key example from Database Functions**

```python
from django.db.models.functions import (
    Concat, Lower, Upper, Length, Left, Right,
    LPad, RPad, Replace, Reverse, StrIndex, Substr, Trim
)
from django.db.models import Value

# Concatenate fields
Person.objects.annotate(
    full_name=Concat('first_name', Value(' '), 'last_name')
)

# Transform case
Person.objects.annotate(
    name_upper=Upper('first_name'),
    name_lower=Lower('last_name')
)

# Get length
Article.objects.annotate(
    title_length=Length('title')
).filter(title_length
```


## Resources

- [Database Functions](https://docs.djangoproject.com/en/6.0/ref/models/database-functions/) — Complete reference of database functions

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*