---
source_course: "django-orm-mastery"
source_lesson: "django-orm-mastery-chainable-queryset-methods"
---

# Chainable QuerySet Methods

## Introduction

The real power of custom QuerySets lies in building a fluent API where methods can be composed in any order. This lesson covers patterns for designing chainable methods that remain readable and maintainable.

## Key Concepts

- **Chainability**: Every custom QuerySet method must return a QuerySet so the next method can operate on it.
- **`as_manager()`**: Converts a custom QuerySet class into a Manager.
- **`from_queryset()`**: Combines a custom Manager with a custom QuerySet.

## Real World Context

Teams build domain-specific query languages using chainable methods. Instead of scattering filter() calls across views, they centralize query logic: `Product.objects.available().on_sale().popular()`.

## Deep Dive

### Designing a Fluent API

A well-designed QuerySet API reads like domain language. Each method returns a QuerySet so calls can be chained in any order:

```python
class ProductQuerySet(models.QuerySet):
    def available(self):
        return self.filter(stock__gt=0, is_active=True)

    def on_sale(self):
        return self.filter(sale_price__lt=F('price'))

    def popular(self, min_orders=10):
        return self.annotate(
            order_count=Count('orderitem')
        ).filter(order_count__gte=min_orders)

    def optimized(self):
        return self.select_related('category', 'brand')

class Product(models.Model):
    objects = ProductQuerySet.as_manager()

# Usage: Product.objects.available().on_sale().popular().optimized()
```

The example above illustrates the pattern in practice. Now let's look at the next approach.

### Combining with from_queryset

Use `from_queryset()` when you need both Manager-level methods (like aggregate statistics) and QuerySet-level chainable methods:

```python
class ProductManager(models.Manager):
    def get_statistics(self):
        return self.aggregate(
            total=Count('id'),
            avg_price=Avg('price')
        )

ProductManager = ProductManager.from_queryset(ProductQuerySet)

class Product(models.Model):
    objects = ProductManager()

# Both work:
Product.objects.available().on_sale()  # QuerySet methods
Product.objects.get_statistics()  # Manager methods
```

## Common Pitfalls

1. **Returning non-QuerySet values** — aggregate() returns a dict, breaking the chain. Put such methods on the Manager, not QuerySet.
2. **Annotation name conflicts** — Two chained methods annotating with the same name overwrite each other.
3. **Forgetting `as_manager()`** — Defining a QuerySet class but using plain `Manager()` means custom methods are inaccessible.

## Best Practices

1. **Name methods like domain concepts** — `available()`, `on_sale()` instead of `filter_active_in_stock()`.
2. **Always return `self.filter(...)` or `self.annotate(...)`** — Ensures chainability.
3. **Create an `optimized()` method** — Bundle select_related/prefetch_related for common view patterns.

## Summary

- Chainable QuerySet methods must always return a QuerySet.
- `as_manager()` exposes QuerySet methods on `Model.objects`.
- `from_queryset()` combines Manager and QuerySet methods.
- Design your query API around domain concepts for readability.

## Code Examples

**Fluent chainable QuerySet API with domain-specific filter methods**

```python
class ProductQuerySet(models.QuerySet):
    def available(self):
        return self.filter(stock__gt=0, is_active=True)

    def on_sale(self):
        return self.filter(sale_price__lt=F('price'))

class Product(models.Model):
    objects = ProductQuerySet.as_manager()

# Chainable: Product.objects.available().on_sale()
```


## Resources

- [Creating a Manager with QuerySet Methods](https://docs.djangoproject.com/en/6.0/topics/db/managers/#creating-a-manager-with-queryset-methods) — Official guide to combining Managers and QuerySets

---

> 📘 *This lesson is part of the [Django ORM Mastery](https://stanza.dev/courses/django-orm-mastery) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*