---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-annotation"
---

# Annotating QuerySets

While `aggregate()` returns a single dictionary, `annotate()` adds computed values to **each object** in a queryset.

## Basic Annotation

```python
from django.db.models import Count, Avg, Sum

# Add article count to each author
authors = Author.objects.annotate(
    article_count=Count('article')
)

for author in authors:
    print(f"{author.name}: {author.article_count} articles")
```

## Annotation vs Aggregation

```python
# Aggregation: One result for entire queryset
Book.objects.aggregate(avg_price=Avg('price'))
# {'avg_price': 25.00}

# Annotation: Value added to each object
books = Book.objects.annotate(price_rank=...)
for book in books:
    print(book.title, book.price_rank)  # Each book has price_rank
```

## Common Annotation Patterns

### Counting Related Objects

```python
# Authors with article count
Author.objects.annotate(
    total_articles=Count('article'),
    published_articles=Count('article', filter=Q(article__is_published=True))
).order_by('-total_articles')

# Categories with product count
Category.objects.annotate(
    product_count=Count('product')
).filter(product_count__gt=0)  # Only non-empty categories
```

### Calculating Totals

```python
# Orders with item totals
Order.objects.annotate(
    item_count=Count('orderitem'),
    subtotal=Sum('orderitem__price')
)

# Customers with total spent
Customer.objects.annotate(
    total_spent=Sum('order__total'),
    order_count=Count('order')
).order_by('-total_spent')
```

### Average Ratings

```python
# Products with average rating
Product.objects.annotate(
    avg_rating=Avg('review__rating'),
    review_count=Count('review')
).filter(
    avg_rating__gte=4.0,
    review_count__gte=10
)
```

## Using F Expressions in Annotations

```python
from django.db.models import F, ExpressionWrapper, DecimalField

# Calculate profit margin
Product.objects.annotate(
    profit=F('price') - F('cost'),
    profit_margin=ExpressionWrapper(
        (F('price') - F('cost')) / F('price') * 100,
        output_field=DecimalField(decimal_places=2, max_digits=5)
    )
)
```

## Conditional Annotations

```python
from django.db.models import Case, When, Value, CharField

# Add status label
Product.objects.annotate(
    stock_status=Case(
        When(stock=0, then=Value('Out of Stock')),
        When(stock__lt=10, then=Value('Low Stock')),
        When(stock__lt=50, then=Value('In Stock')),
        default=Value('Well Stocked'),
        output_field=CharField()
    )
)

# Boolean annotation
from django.db.models import BooleanField

Product.objects.annotate(
    is_low_stock=Case(
        When(stock__lt=10, then=Value(True)),
        default=Value(False),
        output_field=BooleanField()
    )
)
```

## Filtering on Annotations

```python
# Only authors with 5+ articles
Author.objects.annotate(
    article_count=Count('article')
).filter(
    article_count__gte=5
)

# Top-rated products
Product.objects.annotate(
    avg_rating=Avg('review__rating')
).filter(
    avg_rating__gte=4.5
).order_by('-avg_rating')
```

## Ordering by Annotations

```python
# Order by computed value
Author.objects.annotate(
    article_count=Count('article')
).order_by('-article_count')  # Most prolific first

# Order by average rating
Product.objects.annotate(
    avg_rating=Avg('review__rating')
).order_by('-avg_rating')
```

## Performance Tip: values() before annotate()

For grouped annotations:

```python
# Group articles by month
from django.db.models.functions import TruncMonth

Article.objects.annotate(
    month=TruncMonth('pub_date')
).values('month').annotate(
    count=Count('id'),
    total_views=Sum('views')
).order_by('month')

# Result:
# [{'month': datetime(2024, 1, 1), 'count': 15, 'total_views': 5000}, ...]
```

## Resources

- [Annotation Reference](https://docs.djangoproject.com/en/6.0/ref/models/querysets/#annotate) — Official QuerySet.annotate() reference

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*