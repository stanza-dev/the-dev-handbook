---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-q-objects"
---

# Q Objects for Complex Lookups

Q objects allow you to build complex queries with OR, AND, and NOT operations that aren't possible with simple keyword arguments.

## The Problem

With `filter()`, multiple conditions are always ANDed together:

```python
# This is: published=True AND author=user
Article.objects.filter(published=True, author=user)
```

But what about OR conditions?

## Introducing Q Objects

```python
from django.db.models import Q

# OR: published OR featured
Article.objects.filter(Q(published=True) | Q(featured=True))

# AND (explicit)
Article.objects.filter(Q(published=True) & Q(author=user))

# NOT
Article.objects.filter(~Q(status='draft'))
```

## Q Object Operators

| Operator | Meaning | Example |
|----------|---------|----------|
| `|` | OR | `Q(a=1) | Q(b=2)` |
| `&` | AND | `Q(a=1) & Q(b=2)` |
| `~` | NOT | `~Q(a=1)` |

## Complex Query Examples

### Search Implementation

```python
from django.db.models import Q

def search_articles(query):
    """Search in title, body, or author name."""
    return Article.objects.filter(
        Q(title__icontains=query) |
        Q(body__icontains=query) |
        Q(author__name__icontains=query)
    )
```

### Combining with Regular Filters

```python
# Q objects must come before keyword arguments
Article.objects.filter(
    Q(published=True) | Q(author=request.user),
    created_at__year=2024  # Regular filter
)
```

### Dynamic Query Building

```python
def filter_articles(title=None, author=None, min_views=None):
    query = Q()  # Empty Q object
    
    if title:
        query &= Q(title__icontains=title)
    if author:
        query &= Q(author__name=author)
    if min_views:
        query &= Q(views__gte=min_views)
    
    return Article.objects.filter(query)
```

### OR with Multiple Conditions

```python
# Articles that are either:
# - Published AND featured, OR
# - Written by staff members
Article.objects.filter(
    (Q(published=True) & Q(featured=True)) |
    Q(author__is_staff=True)
)
```

### Negation

```python
# All articles EXCEPT drafts
Article.objects.filter(~Q(status='draft'))

# Published articles NOT by specific author
Article.objects.filter(
    Q(published=True) & ~Q(author__username='admin')
)
```

## Building Queries Dynamically

```python
from django.db.models import Q
from functools import reduce
import operator

# Search across multiple fields
search_fields = ['title', 'body', 'summary', 'author__name']
query = 'django'

q_objects = [Q(**{f'{field}__icontains': query}) for field in search_fields]
combined_query = reduce(operator.or_, q_objects)

Article.objects.filter(combined_query)
```

## Common Patterns

### Status-based Filtering

```python
def get_visible_articles(user):
    """Return articles the user can see."""
    if user.is_staff:
        return Article.objects.all()
    
    # Non-staff: published OR own drafts
    return Article.objects.filter(
        Q(published=True) | Q(author=user)
    )
```

### Date Range OR

```python
from datetime import date, timedelta

# Articles from last week OR marked as evergreen
week_ago = date.today() - timedelta(days=7)

Article.objects.filter(
    Q(pub_date__gte=week_ago) | Q(is_evergreen=True)
)
```

## Resources

- [Complex Lookups with Q Objects](https://docs.djangoproject.com/en/6.0/topics/db/queries/#complex-lookups-with-q-objects) — Official documentation on Q objects

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*