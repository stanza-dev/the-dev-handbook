---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-f-expressions"
---

# F Expressions for Field References

F expressions let you reference model field values in queries without loading them into Python. This enables efficient database-level operations.

## Why F Expressions?

Without F expressions (inefficient):

```python
# BAD: Loads all products into Python
for product in Product.objects.all():
    product.price = product.price * 1.1  # 10% increase
    product.save()  # One query per product!
```

With F expressions (efficient):

```python
from django.db.models import F

# GOOD: Single SQL UPDATE statement
Product.objects.update(price=F('price') * 1.1)
```

## Basic Usage

### Updating Based on Current Value

```python
from django.db.models import F

# Increment a counter
Article.objects.filter(pk=1).update(views=F('views') + 1)

# Bulk discount
Product.objects.filter(category='sale').update(
    price=F('price') * 0.9  # 10% off
)
```

### Comparing Fields

```python
# Find products where stock is below reorder level
Product.objects.filter(stock__lt=F('reorder_level'))

# Companies with more employees than chairs
Company.objects.filter(num_employees__gt=F('num_chairs'))

# Articles where updated_at > created_at (has been edited)
Article.objects.filter(updated_at__gt=F('created_at'))
```

### Spanning Relationships

```python
# Products cheaper than their category's average
Product.objects.filter(price__lt=F('category__avg_price'))

# Orders where quantity exceeds product stock
OrderItem.objects.filter(quantity__gt=F('product__stock'))
```

## F with Arithmetic

```python
# Calculate profit margin
Product.objects.annotate(
    profit=F('price') - F('cost')
)

# Double the stock
Product.objects.update(stock=F('stock') * 2)

# Complex calculation
Company.objects.annotate(
    chairs_needed=F('num_employees') - F('num_chairs')
).filter(chairs_needed__gt=0)
```

## F with Dates

```python
from django.db.models import F
from datetime import timedelta

# Find articles not updated in 30 days after creation
Article.objects.filter(
    updated_at__lt=F('created_at') + timedelta(days=30)
)

# Events happening within a week of registration deadline
Event.objects.filter(
    start_date__lt=F('registration_deadline') + timedelta(days=7)
)
```

## Avoiding Race Conditions

F expressions prevent race conditions in concurrent updates:

```python
# Without F (race condition possible)
product = Product.objects.get(pk=1)
product.stock -= 1
product.save()  # Another process might have changed stock!

# With F (atomic operation)
Product.objects.filter(pk=1).update(stock=F('stock') - 1)
```

## F with Order By

```python
# Order by calculated value
Product.objects.order_by(F('price') / F('quantity'))

# Nulls handling
from django.db.models import F

Article.objects.order_by(F('pub_date').desc(nulls_last=True))
Article.objects.order_by(F('pub_date').asc(nulls_first=True))
```

## Combining F with Annotations

```python
from django.db.models import F, ExpressionWrapper, DecimalField

# Calculate percentage
Product.objects.annotate(
    discount_percentage=ExpressionWrapper(
        (F('original_price') - F('price')) / F('original_price') * 100,
        output_field=DecimalField()
    )
)
```

## Important Notes

1. **F expressions are deferred**: The database executes the operation
2. **Refresh after update**: If you need the new value:

```python
Product.objects.filter(pk=1).update(stock=F('stock') - 1)
product.refresh_from_db()  # Get the new stock value
print(product.stock)
```

## Resources

- [F Expressions](https://docs.djangoproject.com/en/6.0/ref/models/expressions/#f-expressions) — Official documentation on F expressions

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*