---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-aggregation-basics"
---

# Aggregation Functions

Aggregation functions compute a single value from a queryset, like counting rows or calculating averages.

## Available Aggregation Functions

```python
from django.db.models import (
    Avg, Count, Max, Min, Sum,
    StdDev, Variance
)
```

| Function | Description |
|----------|-------------|
| `Avg` | Average of values |
| `Count` | Number of items |
| `Max` | Maximum value |
| `Min` | Minimum value |
| `Sum` | Total of values |
| `StdDev` | Standard deviation |
| `Variance` | Statistical variance |

## Basic Aggregation

Use `.aggregate()` to get a dictionary of computed values:

```python
from django.db.models import Avg, Count, Max, Min, Sum

# Single aggregation
result = Book.objects.aggregate(Avg('price'))
# {'price__avg': 34.50}

# Multiple aggregations
result = Book.objects.aggregate(
    avg_price=Avg('price'),
    max_price=Max('price'),
    min_price=Min('price'),
    total_books=Count('id')
)
# {'avg_price': 34.50, 'max_price': 99.99, 'min_price': 9.99, 'total_books': 150}
```

## Count Options

```python
# Count all
Book.objects.aggregate(total=Count('id'))

# Count distinct values
Book.objects.aggregate(unique_authors=Count('author', distinct=True))

# Count with filter (Django 2.0+)
from django.db.models import Count, Q

Publisher.objects.aggregate(
    total_books=Count('book'),
    published_books=Count('book', filter=Q(book__is_published=True)),
    draft_books=Count('book', filter=Q(book__is_published=False))
)
```

## Aggregating Across Relationships

```python
# Average price of books per author
Author.objects.aggregate(avg_book_price=Avg('book__price'))

# Total sales across all orders
Order.objects.aggregate(total_revenue=Sum('orderitem__price'))

# Maximum article views per category
Category.objects.aggregate(max_views=Max('article__views'))
```

## Aggregation with Filtering

```python
# Average price of published books only
Book.objects.filter(
    is_published=True
).aggregate(
    avg_price=Avg('price')
)

# Statistics for a specific category
Book.objects.filter(
    category__name='Fiction'
).aggregate(
    count=Count('id'),
    avg_price=Avg('price'),
    total_pages=Sum('pages')
)
```

## Conditional Aggregation

```python
from django.db.models import Case, When, IntegerField, Sum

Order.objects.aggregate(
    total_orders=Count('id'),
    completed_orders=Count('id', filter=Q(status='completed')),
    pending_orders=Count('id', filter=Q(status='pending')),
    total_revenue=Sum(
        Case(
            When(status='completed', then='amount'),
            default=0,
            output_field=IntegerField()
        )
    )
)
```

## Handling NULL Values

```python
from django.db.models.functions import Coalesce
from django.db.models import Value

# Replace NULL with 0 in aggregation
Book.objects.aggregate(
    avg_rating=Coalesce(Avg('rating'), Value(0.0))
)
```

## Practical Examples

### E-commerce Statistics

```python
from django.db.models import Avg, Count, Sum, F

order_stats = Order.objects.filter(
    created_at__year=2024
).aggregate(
    total_orders=Count('id'),
    total_revenue=Sum('total'),
    avg_order_value=Avg('total'),
    total_items=Sum('orderitem__quantity')
)

print(f"Revenue: ${order_stats['total_revenue']}")
print(f"AOV: ${order_stats['avg_order_value']:.2f}")
```

### Content Statistics

```python
article_stats = Article.objects.aggregate(
    total=Count('id'),
    published=Count('id', filter=Q(status='published')),
    total_views=Sum('views'),
    avg_views=Avg('views'),
    top_views=Max('views')
)
```

## Resources

- [Aggregation](https://docs.djangoproject.com/en/6.0/topics/db/aggregation/) — Official guide to aggregation in Django

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*